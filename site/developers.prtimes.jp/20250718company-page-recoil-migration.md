---
https://developers.prtimes.jp/2025/07/18/company-page-recoil-migration/
---

# 企業ページにおけるRecoilからの段階的移行

こんにちは、フロントエンドエンジニアのやなぎ（ [@apple\_yagi](https://twitter.com/apple_yagi) ）です。

PR TIMESのフロントエンドでは、これまで状態管理にRecoilを利用してきました。しかし、Recoilは現在アーカイブされており、React19にも対応していません。そのため、現在Recoilからの脱却を進めています。昨年、弊社の[桐澤](https://developers.prtimes.jp/author/kirisawakouhei/)が以下のスライドでRecoil脱却の進め方について紹介していましたが、その後の進捗としてRecoilの依存が深いページの脱却を完了することができたのでご紹介します。

目次

## 今回Recoilから脱却したページ

今回は、企業ページの公開ページおよび編集ページにおいてRecoilの利用を廃止しました。公開ページは一般公開されている企業情報の表示ページ、編集ページは企業ユーザーが自社ページを編集するためのページです。以下は公開ページのリンクです。

これらのページでは、公開ページと編集ページのReactコードが密結合しており、Recoilのstateも一元管理されていました。Recoil上で、データフェッチから画面表示用の加工、さらにフォームにおける編集データの管理まで行っていたため、コード量が増え、Recoilのstateも肥大化している状況でした。

## Recoil剥がしの進め方

RecoilからJotaiへの一括移行事例は他社でも多く見かけます。しかし、依存度が高く、かつコードベースの規模が大きい場合は、以下のような理由で一括移行が難しい場合があります。

* バグが発生した時に原因の特定が難しくなる
* Pull Requestが大きくなりすぎ、レビューやQAのコストが高くなる

そのため、段階的な移行が必要不可欠ですが、まずは現状のstate依存関係を把握しなければ移行戦略が立てられませんでした。そこで、[@state-tracer/recoil](https://www.npmjs.com/package/%40state-tracer/recoil) というライブラリを自作し、まず各stateの依存関係の可視化から着手しました。

本稿でいう **依存関係グラフ**（以下グラフ） とは、各 state（atom / selector など）間の依存関係をノードとエッジで表現したものです。矢印の向きは 「依存する側（参照側）」 → 「依存される側（参照される側）」 を示します。これにより、各 state がどこに依存しているか、依存の伝播がどうなっているかを一目で把握でき、大規模アプリの現状把握や移行計画立案に役立ちます。 なお、一般的な データフロー図（データの流れを示す図）とは矢印の意味が逆になる場合があるため注意してください。

@state-tracer/recoil はRecoilのatom、selector、atomFamily、selectorFamily間の依存関係をAST（抽象構文木）から解析し、SVGでグラフ出力します。詳しくは以下の記事でも紹介していますので、あわせてご参照ください。

Zenn

以下は、実際に @state-tracer/recoil で出力した初期状態のグラフです。グラフの右側が依存の根本となるstate、左側が末端のstateとなっています。

このグラフを元に、以下のような手順で徐々にRecoilの依存を排除しました。この手順の対象は下半分の大きなグラフです。

1. グラフの末端にあるselectorを順次 useMemo へ置き換え
2. 公開ページと編集ページそれぞれのstateを分離
3. 公開ページはデータフェッチをTanStack Queryに移行し、Recoilを廃止
4. 編集ページは状態管理をJotaiに移行し、Recoilを廃止

## 末端selectorのuseMemo化

最初のステップとして、グラフの末端に位置するselectorを useMemo に置き換えました。具体的には以下の赤枠のstateが対象となります。

```
const textAtom = atom({
  key: 'text',
  default: '',
});

// 末端のselector
const textLengthSelector = selector({
  key: 'textLength',
  get({get}) {
    const text = get(textAtom);
    return text.length;
  },
});

// selectorをuseMemoに置き換えたhook
const useTextLength = () => {
  const text = useRecoilValue(textAtom);
  const textLength = useMemo(() => {
    return text.length;
  }, [text]);

  return textLength;
};
```

末端のselectorは他のstateから参照されていないため、ほとんどの場合で useMemo に置き換えることができます。末端のselectorを1つずつ useMemo に移行していくことで、まるで玉ねぎの皮を一枚ずつ剥がすように段階的にデータフローの依存関係を減らしていくことができます。これにより、グラフを縮小することができたため、次にご紹介する問題点の発見に繋がりました。

## 公開ページと編集ページのstate分離

useMemo への移行を進めていくと、公開ページと編集ページのstateが結合されていることが問題となりました。企業ページは元々componentなどビューを共通化するためにコードを共有していましたが、stateの共通化までは本来不要でした。そのため、両ページのstate接続点となるselectorを useMemo 化し、stateを分離しました。

before 赤枠が今回削除したstate

after

移行前後のグラフを比べると、移行により1つだった大きなグラフが、公開ページ・編集ページそれぞれのグラフに分離されたことがわかります。この分離によって、以降は独立して各機能ごとに最適なライブラリ選定・移行作業が可能になりました。

## 公開ページのRecoilをTanStack Queryに移行

公開ページではフェッチしたデータを編集することはないため、そもそもRecoilのような状態管理ライブラリは不要と判断し、もともとPR TIMESのフロントエンドで使用していたTanStack Queryへ移行しました。移行対象となるstateは以下の赤枠で囲っている箇所になります。

```
export const getLogoListQuery = selector({
  key: 'logoList',
  async get() {
    return pressKitApi.getPressKitCompaniesCompanyIdLogos({
      companyId: COMPANY_ID,
    });
  },
});

// readonlyLogoListState以外にもreadonlyImageListState、readonlyProfileListStateなどがある
export const readonlyLogoListState = selectorFamily<
  PressKitLogoListByCompanyIdResults[],
  Readonly<FilterParameters>
>({
  key: uuidv4(),
  get:
    (parameters) =>
    ({get}) => {
      const response = get(getLogoListQuery);
      return filterPressKitItem(response.data.results, parameters);
    },
});

export const readonlyLogoListCountState = selectorFamily({
  key: 'readonlyLogoListCount',
  get:
    (selected: SelectedTags) =>
    ({get}) => {
      const logos = get(readonlyLogoListState({selectedTags: selected}));
      const images = get(readonlyImageListState({selectedTags: selected}));
      const profiles = get(readonlyProfileListState({selectedTags: selected}));
      const documents = get(
        readonlyDocumentListState({selectedTags: selected}),
      );
      const guidelineFiles = get(
        readonlyGuidelineFileListState({selectedTags: selected}),
      );

      return {
        logos: logos.length,
        images: images.length,
        guidelineFiles: guidelineFiles.length,
        documents: documents.length,
        profiles: profiles.length,
        pressKit:
          logos.length +
          images.length +
          guidelineFiles.length +
          documents.length +
          profiles.length,
      };
    },
});
```

```
export function useLogoListSuspenseQuery() {
  return useSuspenseQuery({
    queryKey: ['logoList'],
    async queryFn() {
      const response = await pressKitApi.getPressKitCompaniesCompanyIdLogos({
        companyId: COMPANY_ID,
      });
      return response.data;
    },
    // Recoilのselectorと挙動を合わせるため、明示的にフェッチしない限り再フェッチはしないようにする
    gcTime: Infinity,
    staleTime: Infinity,
    refetchOnMount: false,
    refetchOnWindowFocus: false,
    refetchOnReconnect: false,
    refetchInterval: false,
  });
}

export function useReadonlyProfileListSuspenseQuery(
  parameters: FilterParameters,
) {
  const {data} = useProfileListSuspenseQuery();
  const filteredData = filterPressKitItem(data.results, parameters);

  return {
    data: filteredData,
  };
}

export function useReadonlyPressKitCountSuspenseQuery(
  selectedTags: SelectedTags,
) {
  const {data: logos} = useReadonlyLogoListSuspenseQuery({selectedTags});
  const {data: images} = useReadonlyImageListSuspenseQuery({selectedTags});
  const {data: profiles} = useReadonlyProfileListSuspenseQuery({selectedTags});
  const {data: documents} = useReadonlyDocumentListSuspenseQuery({
    selectedTags,
  });
  const {data: guidelineFiles} = useReadonlyGuidelineFileListSuspenseQuery({
    selectedTags,
  });

  return {
    logos: logos.length,
    images: images.length,
    guidelineFiles: guidelineFiles.length,
    documents: documents.length,
    profiles: profiles.length,
    pressKit:
      logos.length +
      images.length +
      guidelineFiles.length +
      documents.length +
      profiles.length,
  };
}
```

Recoilのselectorで非同期を扱う際にはuseRecoilStateLoadableなどのloadable hookまたは、Suspenseで待機するという方法があります。今回のページではSuspenseで待機していたため、そのSuspense境界をそのまま活用し、TanStack QueryのuseSuspenseQueryを用いて同期的にデータにアクセスしています。

TanStack Queryへの移行により、グラフはシンプルになり、残る大きな箇所は編集ページのRecoilのみとなりました。

before

after

## 編集ページのRecoilをJotaiに移行する

編集ページでは、Recoilの移行先としてJotaiを採用しました。理由としては、RecoilとAPIが近く、変更量を最小限に抑えやすかった点や、atomFamily/selectorFamilyの利用箇所が多かった点が挙げられます。

一方で、JotaiにはRecoilと異なるRead-only atomやWrite-only atomなどの概念もあり、単純に recoil を jotai へimportし直すだけでは移行できませんでした。工夫した点や苦労したポイントについては、具体例を挙げながらご紹介します。

```
export const pressKitQuery = selector({
  key: uuidv4(),
  async get() {
    return pressKitAdminApi.getPressKit();
  },
});

export const editableLogoDetailState = atomFamily<
  PressKitLogoEditable | undefined,
  string
>({
  key: uuidv4(),
  default: selectorFamily({
    key: uuidv4(),
    get:
      (id: string) =>
      ({get}) => {
        const response = get(pressKitQuery);
        const item = response.data.logos.find((item) => item.id === id);
        if (item) {
          const tags = [];
          for (const tag of item.tags) {
            const itemTag = get(editablePressKitTagDetailState(tag.id));
            if (itemTag) {
              tags.push(itemTag);
            }
          }

          return {
            objectId: getDummyId(),
            tags,
          };
        }

        return undefined;
      },
  }),
});

export const editableLogoIdListState = atom({
  key: uuidv4(),
  default: selector({
    key: uuidv4(),
    get({get}) {
      const response = get(pressKitQuery);
      return response.data.logos.map((item) => item.id);
    },
  }),
});
```

```
export const pressKitQuery = atom<PressKitOverallData>(
  // storeで取得したデータを初期値にするため、デフォルトでは空オブジェクトを設定している
  {} as PressKitOverallData,
);

// atomFamily/selectorFamilyの置き換えパターン
export const editableLogoDetailState = atomFamily((id: string) =>
  atomWithDefault((get) => {
    const response = get(pressKitQuery);
    const item = response.logos.find((item) => item.id === id);
    if (item) {
      const tags = [];
      for (const tag of item.tags) {
        const itemTag = get(editablePressKitTagDetailState(tag.id));
        if (itemTag) {
          tags.push(itemTag);
        }
      }

      return {
        objectId: getDummyId(),
        tags,
      };
    }

    return undefined;
  }),
);

// atom/default selectorの置き換えパターン
export const editableLogoIdListState = atomWithDefault((get) => {
  const response = get(pressKitQuery);
  return response.logos.map((item) => item.id);
});
```

Recoilの atom/selector やatomFamily/selectorFamily は、Jotaiの atomWithDefault や atomFamily/atomWithDefault で比較的スムーズに移行できました。ただし、ひとつ pressKitQuery の扱いについては変更しています。

pressKitQueryで非同期データをstateで保持しようとすると、 依存先のstate（editableLogoDetailState や editableLogoIdListState）の型がPromiseになるため、Recoilと同じようにuseRecoilCallbackによる遅延取得が難しくなります。

```
export function useGetCount() {
  const getCount = useRecoilCallback(({snapshot}) => () => {
    const loadable = snapshot.getLoadable(editableLogoIdListState);
    if (loadable.state === 'hasValue') {
      return loadable.contents.length;
    }

    // eslint-disable-next-line @typescript-eslint/no-unused-expressions
    undefined;
  });

  return getCount;
}

export function Test() {
  const {getCount} = useGetCount();
  const count = getCount();

  return <div>{count}</div>;
}
```

```
export function useGetCount() {
  const getCount = useAtomCallback(
    useCallback(async (get) => {
      const list = await get(editableProfileIdListState);
      return list.length;
    }, []),
  );

  return getCount;
}

export function Test() {
  const {getCount} = useGetCount();
  // countがPromise<number>となり、awaitする必要がある
  const count = getCount();

  return <div>{count}</div>;
}
```

また、selector内では他の非同期なstateを同期的に取得できたりと、RecoilとJotaiでは非同期データの扱い方に大きな違いがありました。そのため、Jotai移行の方針としてJotai内では非同期を扱わないこととしました。データフェッチが必要な場合は、以下のようにTanStack Queryで行い、値をStore Provider経由でJotai stateにセットする形に切り替えています。

```
import {createStore, Provider} from 'jotai';

const store = createStore();

export function PressKitEditJotaiProvider({
  children,
}: {
  readonly children: ReactNode;
}) {
  const {data, isLoading, isError} = useQuery({
    queryKey: ['pressKit'],
    async queryFn() {
      const response = await pressKitAdminApi.getPressKit();

      return response.data;
    },
    refetchOnWindowFocus: false,
    refetchOnReconnect: false,
    refetchOnMount: false,
    retry: false,
    gcTime: Infinity,
    staleTime: Infinity,
  });

  if (isError) return <Error />;

  if (isLoading || !data) return <Loading />;

  store.set(pressKitQuery, data);

  return <Provider store={store}>{children}</Provider>;
}
```

これにより、Jotai内で非同期が発生せず、Recoilからの移行をスムーズに行うことができました。こちらはJotai移行後のグラフになります。

before

after

残っているstateは依存関係が浅いため、useMemoやuseContextに移行していき、企業ページの全てのRecoil stateを排除することができました。

Jotai移行後のJotaiの依存関係グラフはこのようにシンプルな状態を保っています。

@state-tracer/recoil
のJotai版である [@state-tracer/jotai](https://www.npmjs.com/package/%40state-tracer/jotai) で出力しています

## まとめ

企業ページのRecoil剥がしでは @state-tracer/recoil を用いてデータフローの可視化から着手し、段階的なRecoilからの移行を行いました。

また、state の性質や依存の深さに応じて、計算コスト削減が目的のものは useMemo、アプリ全体で共有したい軽量 state は useContext、サーバー同期が必要な領域は TanStack Query、細粒度で分割したいローカル state は Jotai へと置き換え、特定ライブラリへの依存を排除しました。その結果、今後さらにライブラリの移行が必要になっても柔軟に対応できる構成を実現できたと考えています。

今回で企業ページのRecoil剥がしは完了しましたが、今後はさらに大規模なエディター機能のRecoil剥がしが控えています。今回学んだことを糧に安全に移行しきりたいと思います。

エディターの依存関係グラフ

## We are hiring!

フロントエンドエンジニアを含む各種ポジションで採用を進めています！興味があればぜひご応募ください。

株式会社PR TIMES

株式会社PR TIMESでは現在03-4. 開発部 フロントエンドエンジニアを募集しています。

株式会社PR TIMES

株式会社PR TIMESが公開している、02．開発部 の求人一覧です
