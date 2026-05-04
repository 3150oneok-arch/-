<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  <meta name="theme-color" content="#0a0a1a" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
  <meta name="apple-mobile-web-app-title" content="TAXI MANAGER" />
  <title>TAXI MANAGER</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { background: #0a0a1a; -webkit-tap-highlight-color: transparent; font-family: "Hiragino Sans", "Noto Sans JP", sans-serif; }
    input, select, button { font-family: inherit; }
    input[type=number]::-webkit-inner-spin-button,
    input[type=number]::-webkit-outer-spin-button { opacity: 1; }
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- React CDN -->
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  <!-- Babel for JSX -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <script type="text/babel">
    const { useState, useEffect, useRef } = React;


// ===== ストレージヘルパー（localStorage）=====
const storage = {
  async get(key) {
    try { return localStorage.getItem(key); } catch { return null; }
  },
  async set(key, value) {
    try { localStorage.setItem(key, value); } catch {}
  },
};



// ===== 交通圏マスタ（全国版）=====

// 都道府県グループ
const PREFECTURES = [
  { id: "tokyo",     name: "東京都" },
  { id: "kanagawa", name: "神奈川県" },
  { id: "chiba",    name: "千葉県" },
  { id: "saitama",  name: "埼玉県" },
  { id: "osaka",    name: "大阪府" },
  { id: "kyoto",    name: "京都府" },
  { id: "hyogo",    name: "兵庫県" },
  { id: "aichi",    name: "愛知県" },
  { id: "fukuoka",  name: "福岡県" },
];

const ALL_ZONES = {

  // ========== 東京都 ==========
  tokubestu: {
    id: "tokubestu", pref: "tokyo",
    name: "特別区・武三交通圏", short: "特別区・武三",
    desc: "東京23区・武蔵野市・三鷹市",
    peakNote: "深夜の繁華街需要が最強。終電後の新宿・六本木・渋谷に集中。",
    taxiMeter: "初乗り500円（1.096km）以降237mごと100円",
    restriction: "23区内は自由営業。武蔵野・三鷹はABC制限あり。",
    airports: [
      { name: "羽田空港", code: "HND", lines: "京急・モノレール", tip: "第3T国際線の深夜が狙い目。" },
      { name: "成田空港", code: "NRT", lines: "成田エクスプレス・スカイライナー", tip: "高単価だが帰路は空回送リスクあり。" },
    ],
    areas: [
      { id: "t1", name: "新宿・歌舞伎町",  grade: "S", peak: "金土深夜",    tips: "歌舞伎町0〜3時が最強。西新宿ビル街は18〜20時安定。", lat: 35.6938, lng: 139.7036 },
      { id: "t2", name: "渋谷・恵比寿",    grade: "S", peak: "金土22時〜",  tips: "スクランブル周辺待機が鉄板。恵比寿は高単価多め。", lat: 35.6580, lng: 139.7016 },
      { id: "t3", name: "六本木・赤坂",    grade: "S", peak: "毎日深夜",    tips: "ヒルズ・ミッドタウン周辺は長距離多し。外国人客も狙える。", lat: 35.6628, lng: 139.7315 },
      { id: "t4", name: "銀座・有楽町",    grade: "A", peak: "平日18〜21時", tips: "クラブ帰り高単価あり。百貨店閉店後の客も多い。", lat: 35.6721, lng: 139.7649 },
      { id: "t5", name: "品川・天王洲",    grade: "A", peak: "平日朝夕",    tips: "新幹線需要。ビジネス客の羽田行きが稼ぎやすい。", lat: 35.6284, lng: 139.7387 },
      { id: "t6", name: "池袋",            grade: "A", peak: "金土深夜",    tips: "東西両口ともに深夜需要大。郊外行き長距離が出る。", lat: 35.7295, lng: 139.7109 },
      { id: "t7", name: "浅草・上野",      grade: "B", peak: "昼間・休日",  tips: "観光客多め。空港行きを狙う戦略が有効。", lat: 35.7147, lng: 139.7967 },
      { id: "t8", name: "秋葉原・神田",    grade: "B", peak: "平日昼・夕",  tips: "ビジネス短距離多め。イベント時は跳ねる。", lat: 35.6984, lng: 139.7731 },
      { id: "t9", name: "東京駅・丸の内",  grade: "A", peak: "平日朝夕・深夜", tips: "新幹線・長距離バス連絡。高単価の郊外行き多い。", lat: 35.6812, lng: 139.7671 },
      { id: "t10",name: "豊洲・有明",      grade: "B", peak: "イベント時",   tips: "ビッグサイト・豊洲PIT時が勝負。平常時は薄い。", lat: 35.6453, lng: 139.7940 },
    ],
    stations: [
      { name: "新宿",  lines: "JR各線・小田急・京王・地下鉄", lastTrain: "約0:30", tip: "終電後のタクシー需要が最大。" },
      { name: "渋谷",  lines: "JR・東横・田園都市・地下鉄",   lastTrain: "約0:30", tip: "ハチ公口周辺が拾いやすい。" },
      { name: "池袋",  lines: "JR各線・丸ノ内・有楽町・副都心",lastTrain: "約0:30", tip: "東西両口で客層が違う。" },
      { name: "品川",  lines: "JR各線・京急",                 lastTrain: "約0:25", tip: "新幹線最終接続客を狙う。" },
      { name: "東京",  lines: "JR各線・地下鉄各線",           lastTrain: "約0:30", tip: "丸の内口タクシー乗り場が穴場。" },
      { name: "六本木",lines: "日比谷線・大江戸線",            lastTrain: "約0:14", tip: "大江戸線最終後は需要急増。" },
    ],
    venueIds: ["ariake_arena","bigsight","national_olympic","ajinomoto","budokan","marunouchi","toyosu_pit","zepp_haneda","other"],
  },

  kita_tama: {
    id: "kita_tama", pref: "tokyo",
    name: "北多摩交通圏", short: "北多摩",
    desc: "立川・八王子・府中・調布・小平・東村山など",
    peakNote: "郊外型。イベント終了後の駅前待機と帰宅需要が中心。",
    taxiMeter: "初乗り500円（1.096km）以降237mごと100円",
    restriction: "23区内への乗り入れ可。区内での客扱いは不可。",
    airports: [
      { name: "羽田空港", code: "HND", lines: "京急・モノレール", tip: "調布・府中方面からの需要で高単価になりやすい。" },
    ],
    areas: [
      { id: "k1", name: "立川駅周辺",   grade: "A", peak: "平日夕・金土深夜", tips: "南口乗り場が主戦場。昭島・福生方面が出やすい。", lat: 35.6985, lng: 139.4131 },
      { id: "k2", name: "八王子駅周辺", grade: "A", peak: "金土深夜",         tips: "北口飲み街から郊外長距離が出る。", lat: 35.6558, lng: 139.3239 },
      { id: "k3", name: "府中・調布",   grade: "B", peak: "イベント時",        tips: "味スタ試合日は圧倒的需要。平常時は住宅地送迎。", lat: 35.6702, lng: 139.4779 },
      { id: "k4", name: "吉祥寺・三鷹", grade: "B", peak: "週末昼〜夜",       tips: "飲食帰り中距離多め。深夜の終電後も需要あり。", lat: 35.7024, lng: 139.5796 },
    ],
    stations: [
      { name: "立川",   lines: "JR中央線・南武線・多摩モノレール", lastTrain: "約0:30", tip: "南口乗り場が拾いやすい。" },
      { name: "八王子", lines: "JR中央線・横浜線・八高線",         lastTrain: "約0:20", tip: "北口飲み街からの郊外長距離が主力。" },
      { name: "府中・調布", lines: "京王線",                      lastTrain: "約0:20", tip: "味スタ試合後は別格。" },
      { name: "吉祥寺", lines: "JR中央線・京王井の頭線",           lastTrain: "約0:30", tip: "週末夜の飲食帰り需要が安定。" },
    ],
    venueIds: ["ajinomoto","seibu_dome","other"],
  },

  minami_tama: {
    id: "minami_tama", pref: "tokyo",
    name: "南多摩交通圏", short: "南多摩",
    desc: "町田・多摩・稲城・日野など",
    peakNote: "住宅地・郊外型。駅前帰宅需要と平日ビジネス需要が中心。",
    taxiMeter: "初乗り500円（1.096km）以降237mごと100円",
    restriction: "23区内への乗り入れ可。区内での客扱いは不可。",
    airports: [{ name: "羽田空港", code: "HND", lines: "京急・モノレール", tip: "町田方面からは距離があるが高単価。" }],
    areas: [
      { id: "m1", name: "町田駅周辺",   grade: "A", peak: "金土深夜", tips: "相模原方面への長距離も出る。南口飲み街を狙う。", lat: 35.5414, lng: 139.4457 },
      { id: "m2", name: "多摩センター", grade: "B", peak: "平日夕",   tips: "住宅地帰宅需要中心。モール閉店後に需要あり。", lat: 35.6369, lng: 139.4434 },
    ],
    stations: [
      { name: "町田",      lines: "JR横浜線・小田急線",                       lastTrain: "約0:25", tip: "神奈川方面需要も取り込める立地。" },
      { name: "多摩センター", lines: "京王線・小田急線・多摩モノレール",       lastTrain: "約0:20", tip: "住宅地送迎が基本。深夜は薄め。" },
    ],
    venueIds: ["other"],
  },

  nishi_tama: {
    id: "nishi_tama", pref: "tokyo",
    name: "西多摩交通圏", short: "西多摩",
    desc: "青梅・奥多摩・あきる野・福生など",
    peakNote: "需要が薄く長距離主体。観光シーズン・祭り時に集中する。",
    taxiMeter: "初乗り500円（1.096km）以降237mごと100円",
    restriction: "23区内への乗り入れ可。区内での客扱いは不可。",
    airports: [{ name: "羽田空港", code: "HND", lines: "京急・モノレール", tip: "福生・拝島方面からは比較的アクセスしやすい。" }],
    areas: [
      { id: "w1", name: "青梅・河辺", grade: "B", peak: "週末・祭り時",     tips: "青梅マラソン・花火大会時が最大の稼ぎ時。", lat: 35.7879, lng: 139.2757 },
      { id: "w2", name: "福生・拝島", grade: "B", peak: "米軍基地イベント時", tips: "横田基地フレンドシップデー等は別格の需要。", lat: 35.7387, lng: 139.3289 },
    ],
    stations: [
      { name: "青梅", lines: "JR青梅線",                              lastTrain: "約23:30", tip: "終電が早い。長距離になりやすい。" },
      { name: "拝島", lines: "JR青梅線・五日市線・八高線・西武線",    lastTrain: "約0:10",  tip: "乗換駅。拝島以西の帰宅需要あり。" },
    ],
    venueIds: ["other"],
  },

  // ========== 神奈川県 ==========
  yokohama: {
    id: "yokohama", pref: "kanagawa",
    name: "横浜・川崎交通圏", short: "横浜・川崎",
    desc: "横浜市・川崎市・三浦市など",
    peakNote: "みなとみらい・関内の深夜需要が主力。川崎はナイター後が稼ぎ時。",
    taxiMeter: "初乗り500円（1.052km）以降233mごと100円",
    restriction: "横浜・川崎市内は自由営業。東京23区への乗り入れ可。",
    airports: [
      { name: "羽田空港", code: "HND", lines: "京急・モノレール", tip: "横浜からは高単価。川崎経由も狙い目。" },
    ],
    areas: [
      { id: "y1", name: "みなとみらい・馬車道", grade: "S", peak: "金土深夜",   tips: "ランドマーク周辺は深夜の高単価需要が集中。終電後は独壇場。", lat: 35.4561, lng: 139.6381 },
      { id: "y2", name: "関内・伊勢佐木町",     grade: "A", peak: "毎日深夜",   tips: "飲み街として需要安定。中区内の長距離も出やすい。", lat: 35.4437, lng: 139.6430 },
      { id: "y3", name: "横浜駅周辺",           grade: "A", peak: "平日夕・深夜", tips: "東口・西口ともに需要安定。新幹線接続客を狙う。", lat: 35.4657, lng: 139.6223 },
      { id: "y4", name: "川崎駅周辺",           grade: "A", peak: "平日夕・金土", tips: "ラゾーナ帰り・飲み客多め。都内への長距離が出やすい。", lat: 35.5308, lng: 139.6991 },
      { id: "y5", name: "溝の口・二子玉川",     grade: "B", peak: "週末夕〜深夜", tips: "田園都市線終電後の帰宅需要。住宅地送迎が多い。", lat: 35.5846, lng: 139.6034 },
    ],
    stations: [
      { name: "横浜",     lines: "JR各線・東横・みなとみらい・京急・相鉄", lastTrain: "約0:30", tip: "東口タクシー乗り場が主戦場。" },
      { name: "関内",     lines: "JR根岸線・市営地下鉄",                  lastTrain: "約0:20", tip: "飲み街直結。深夜需要が安定している。" },
      { name: "川崎",     lines: "JR各線・京急",                          lastTrain: "約0:30", tip: "東口・西口両方に需要。都内長距離も出る。" },
      { name: "新横浜",   lines: "JR横浜線・新幹線・市営地下鉄・東横",    lastTrain: "約0:20", tip: "新幹線最終後の需要を狙う。" },
    ],
    venueIds: ["pia_arena","nissan_stadium","yokohama_arena","other"],
  },

  kanagawa_other: {
    id: "kanagawa_other", pref: "kanagawa",
    name: "神奈川県その他交通圏", short: "神奈川その他",
    desc: "相模原・藤沢・小田原・平塚など",
    peakNote: "郊外住宅地需要中心。相模大野・藤沢の深夜飲み需要が主力。",
    taxiMeter: "初乗り500円（1.052km）以降233mごと100円",
    restriction: "各市内で自由営業。他交通圏へのまたぎ乗り入れは規制あり。",
    airports: [{ name: "羽田空港", code: "HND", lines: "京急", tip: "藤沢・相模原方面から高単価になりやすい。" }],
    areas: [
      { id: "kn1", name: "相模大野・相模原", grade: "A", peak: "金土深夜", tips: "小田急ターミナル。深夜の帰宅需要で安定。長距離多め。", lat: 35.5442, lng: 139.4515 },
      { id: "kn2", name: "藤沢・辻堂",       grade: "A", peak: "金土深夜", tips: "湘南エリアの飲み需要。海沿い客は高単価になりやすい。", lat: 35.3383, lng: 139.4897 },
      { id: "kn3", name: "平塚・茅ヶ崎",     grade: "B", peak: "イベント時", tips: "サザンビーチ等の夏季イベント時が最大の稼ぎ時。", lat: 35.3275, lng: 139.3500 },
    ],
    stations: [
      { name: "相模大野", lines: "小田急線",       lastTrain: "約0:20", tip: "急行・快速急行の終着駅。帰宅需要が集中。" },
      { name: "藤沢",     lines: "JR東海道線・小田急・江ノ電", lastTrain: "約0:25", tip: "湘南エリアのハブ。深夜の飲み需要が安定。" },
      { name: "平塚",     lines: "JR東海道線",     lastTrain: "約0:20", tip: "湘南方面の帰宅需要あり。" },
    ],
    venueIds: ["other"],
  },

  // ========== 千葉県 ==========
  chiba_city: {
    id: "chiba_city", pref: "chiba",
    name: "千葉交通圏", short: "千葉",
    desc: "千葉市・習志野・市川・船橋・松戸・浦安など",
    peakNote: "舞浜・幕張の大型施設需要と深夜の繁華街需要が二本柱。",
    taxiMeter: "初乗り500円（1.096km）以降237mごと100円",
    restriction: "千葉県内は自由営業。東京23区への乗り入れ可、区内での客扱いは不可。",
    airports: [
      { name: "成田空港", code: "NRT", lines: "成田エクスプレス・スカイライナー", tip: "都内客の乗車機会あり。帰路は空回送に注意。" },
      { name: "羽田空港", code: "HND", lines: "京急・モノレール", tip: "浦安・市川方面からは比較的需要あり。" },
    ],
    areas: [
      { id: "c1", name: "舞浜・浦安",     grade: "S", peak: "毎日深夜〜閉園後", tips: "TDR閉園後（22時前後）が最大。駐車場から帰宅客を狙う。", lat: 35.6329, lng: 139.8808 },
      { id: "c2", name: "幕張新都心",     grade: "A", peak: "イベント時・平日夕", tips: "幕張メッセ・ZOZOマリン近接。ホテル客も稼ぎやすい。", lat: 35.6438, lng: 140.0341 },
      { id: "c3", name: "千葉駅周辺",     grade: "A", peak: "平日夕・金土深夜", tips: "ペリエ・東口の飲み街から帰宅需要。長距離になりやすい。", lat: 35.6137, lng: 140.1133 },
      { id: "c4", name: "船橋・津田沼",   grade: "A", peak: "金土深夜",         tips: "東武・JRのターミナル。終電後の帰宅需要が安定。", lat: 35.6947, lng: 140.0182 },
      { id: "c5", name: "本八幡・市川",   grade: "B", peak: "平日夕・金土",     tips: "都内に近くビジネス客需要あり。都内行き長距離も出やすい。", lat: 35.7252, lng: 139.9246 },
    ],
    stations: [
      { name: "千葉",   lines: "JR総武線・京葉線・内房線・外房線・地下鉄", lastTrain: "約0:25", tip: "東口飲み街からの帰宅需要が主力。" },
      { name: "船橋",   lines: "JR総武線・東武野田線・京成線",             lastTrain: "約0:25", tip: "終電後の帰宅需要が安定している。" },
      { name: "海浜幕張", lines: "JR京葉線",                                lastTrain: "約0:05", tip: "終電が早い。イベント終了後は独占状態になりやすい。" },
      { name: "舞浜",   lines: "JR京葉線",                                  lastTrain: "約0:10", tip: "TDR閉園後は圧倒的需要。終電前後が稼ぎ時。" },
    ],
    venueIds: ["makuhari","zozomarineplus","other"],
  },

  // ========== 埼玉県 ==========
  saitama: {
    id: "saitama", pref: "saitama",
    name: "埼玉交通圏", short: "埼玉",
    desc: "さいたま市・川口・越谷・所沢・川越など",
    peakNote: "大宮・浦和の深夜飲み需要と、スタジアム・アリーナのイベント需要が柱。",
    taxiMeter: "初乗り500円（1.096km）以降237mごと100円",
    restriction: "埼玉県内は自由営業。東京23区への乗り入れ可、区内での客扱いは不可。",
    airports: [{ name: "羽田空港", code: "HND", lines: "京浜急行", tip: "大宮方面からは距離があるが高単価になる。" }],
    areas: [
      { id: "s1", name: "大宮駅周辺",   grade: "S", peak: "金土深夜",    tips: "東口・西口ともに飲み需要大。新幹線接続客も狙える。", lat: 35.9063, lng: 139.6233 },
      { id: "s2", name: "浦和・さいたま", grade: "A", peak: "平日夕・金土", tips: "さいたまスーパーアリーナ直近。イベント時は別格。", lat: 35.8577, lng: 139.6455 },
      { id: "s3", name: "川越駅周辺",   grade: "B", peak: "週末夕〜深夜", tips: "小江戸観光帰りと飲み需要。川越まつり時期は特需。", lat: 35.9234, lng: 139.4853 },
      { id: "s4", name: "川口・蕨",     grade: "B", peak: "平日夕",       tips: "都内への通勤客多め。帰宅需要が安定している。", lat: 35.8077, lng: 139.7229 },
    ],
    stations: [
      { name: "大宮",   lines: "JR各線・新幹線・埼玉新都市交通",      lastTrain: "約0:30", tip: "新幹線最終後も需要あり。東口が主戦場。" },
      { name: "浦和",   lines: "JR京浜東北線・高崎線・宇都宮線",      lastTrain: "約0:25", tip: "さいたまSSAへの需要あり。" },
      { name: "川越",   lines: "JR川越線・東武東上線・西武新宿線",    lastTrain: "約0:15", tip: "観光地需要と終電後帰宅需要が合わさる。" },
    ],
    venueIds: ["saitama_super_arena","seibu_dome","other"],
  },

  // ========== 大阪府 ==========
  osaka_city: {
    id: "osaka_city", pref: "osaka",
    name: "大阪市域交通圏", short: "大阪市域",
    desc: "大阪市・堺市・守口市・東大阪市など",
    peakNote: "ミナミ（道頓堀・難波）の深夜需要が最強。キタ（梅田）も安定。",
    taxiMeter: "初乗り500円（1.05km）以降264mごと100円",
    restriction: "大阪市域内は自由営業。兵庫・京都への乗り入れ可。",
    airports: [
      { name: "関西空港", code: "KIX", lines: "南海・はるか", tip: "都心から高単価だが帰路は空回送リスクあり。" },
      { name: "伊丹空港", code: "ITM", lines: "リムジンバス", tip: "梅田・難波方面からアクセス良好で需要安定。" },
    ],
    areas: [
      { id: "o1", name: "道頓堀・難波",   grade: "S", peak: "毎日深夜",    tips: "ミナミの中心。0〜3時が最強。外国人観光客も多く高単価あり。", lat: 34.6687, lng: 135.5019 },
      { id: "o2", name: "梅田・北新地",   grade: "S", peak: "平日・金土深夜", tips: "北新地のクラブ帰りは超高単価。梅田ターミナル需要も安定。", lat: 34.7024, lng: 135.4959 },
      { id: "o3", name: "心斎橋・四ツ橋", grade: "A", peak: "毎日夜〜深夜", tips: "飲み街として安定。ミナミとキタの中間で流しやすい。", lat: 34.6769, lng: 135.4986 },
      { id: "o4", name: "天王寺・阿倍野", grade: "A", peak: "平日夕・週末",  tips: "ハルカス周辺の買い物帰り。あべのベルタ周辺が拾いやすい。", lat: 34.6464, lng: 135.5136 },
      { id: "o5", name: "新大阪",         grade: "A", peak: "新幹線最終前後", tips: "新幹線の最終後は需要急増。北口タクシー乗り場が主戦場。", lat: 34.7333, lng: 135.5000 },
      { id: "o6", name: "本町・淀屋橋",   grade: "B", peak: "平日夕",       tips: "ビジネス街。17〜19時の帰宅ラッシュに集中する。", lat: 34.6876, lng: 135.5013 },
    ],
    stations: [
      { name: "梅田（大阪）", lines: "JR・阪急・阪神・地下鉄各線", lastTrain: "約0:30", tip: "日本有数のターミナル。終電後は需要爆発。" },
      { name: "難波",         lines: "南海・近鉄・地下鉄各線",     lastTrain: "約0:25", tip: "ミナミの中心。深夜の需要が最大。" },
      { name: "天王寺",       lines: "JR・近鉄・地下鉄各線",       lastTrain: "約0:25", tip: "南大阪のハブ。買い物・飲み客が集まる。" },
      { name: "新大阪",       lines: "JR新幹線・御堂筋線",         lastTrain: "約0:20", tip: "新幹線最終後は需要集中。" },
    ],
    venueIds: ["kyocera_dome","intex_osaka","namba_city","other"],
  },

  // ========== 京都府 ==========
  kyoto: {
    id: "kyoto", pref: "kyoto",
    name: "京都交通圏", short: "京都",
    desc: "京都市・宇治市・亀岡市など",
    peakNote: "観光需要が主力。紅葉・桜シーズンと深夜の祇園が最大の稼ぎ時。",
    taxiMeter: "初乗り590円（1.1km）以降270mごと100円（やや高め）",
    restriction: "京都府内は自由営業。大阪・滋賀への乗り入れ可。",
    airports: [{ name: "伊丹空港", code: "ITM", lines: "リムジンバス", tip: "京都市内からは高単価になりやすい。" }],
    areas: [
      { id: "ky1", name: "祇園・河原町",   grade: "S", peak: "毎日深夜・観光シーズン", tips: "花見小路周辺の深夜が最高単価。外国人観光客が多く英語対応が有利。", lat: 35.0040, lng: 135.7756 },
      { id: "ky2", name: "京都駅周辺",     grade: "A", peak: "毎日夕〜深夜",          tips: "新幹線・在来線の接続需要。深夜は空港行きも狙える。", lat: 34.9857, lng: 135.7589 },
      { id: "ky3", name: "嵐山・嵯峨野",   grade: "A", peak: "観光シーズン昼間",       tips: "観光地帰りの長距離が出やすい。シーズン外は薄い。", lat: 35.0094, lng: 135.6772 },
      { id: "ky4", name: "烏丸・四条",     grade: "B", peak: "平日夕・週末夜",         tips: "ビジネス街と繁華街の中間。需要は安定しているが単価は低め。", lat: 35.0033, lng: 135.7588 },
    ],
    stations: [
      { name: "京都",   lines: "JR各線・新幹線・近鉄・地下鉄", lastTrain: "約0:20", tip: "八条口・烏丸口両方に需要。新幹線最終後が特需。" },
      { name: "河原町", lines: "阪急京都線",                   lastTrain: "約0:20", tip: "祇園・繁華街直結。深夜の飲み客を狙う。" },
    ],
    venueIds: ["other"],
  },

  // ========== 兵庫県 ==========
  kobe: {
    id: "kobe", pref: "hyogo",
    name: "神戸交通圏", short: "神戸",
    desc: "神戸市・明石市・西宮市・尼崎市など",
    peakNote: "三宮・元町の深夜需要と新幹線接続需要が主力。",
    taxiMeter: "初乗り600円（1.2km）以降236mごと100円",
    restriction: "兵庫県内は自由営業。大阪・京都への乗り入れ可。",
    airports: [{ name: "伊丹空港", code: "ITM", lines: "リムジンバス", tip: "神戸市内から高単価になりやすい。" }],
    areas: [
      { id: "kb1", name: "三宮・元町",   grade: "S", peak: "金土深夜",    tips: "神戸最大の繁華街。深夜の飲み帰りが最大需要。", lat: 34.6913, lng: 135.1956 },
      { id: "kb2", name: "新神戸・三宮", grade: "A", peak: "新幹線最終後", tips: "新幹線接続需要。郊外方面への長距離が出やすい。", lat: 34.7054, lng: 135.1930 },
      { id: "kb3", name: "尼崎・西宮",   grade: "B", peak: "平日夕",       tips: "大阪との境界。大阪方面への需要もある。", lat: 34.7325, lng: 135.4069 },
    ],
    stations: [
      { name: "三宮",   lines: "JR・阪急・阪神・地下鉄・ポートライナー", lastTrain: "約0:25", tip: "神戸の中心。終電後の需要が最大。" },
      { name: "新神戸", lines: "JR新幹線・地下鉄",                       lastTrain: "約0:15", tip: "新幹線最終後の需要を狙う。" },
    ],
    venueIds: ["other"],
  },

  // ========== 愛知県 ==========
  nagoya: {
    id: "nagoya", pref: "aichi",
    name: "名古屋交通圏", short: "名古屋",
    desc: "名古屋市・豊田市・岡崎市・春日井市など",
    peakNote: "栄・錦の深夜需要が核心。名古屋駅の新幹線接続需要も重要。",
    taxiMeter: "初乗り500円（1.177km）以降252mごと100円",
    restriction: "名古屋市域内は自由営業。愛知県内各地への乗り入れ可。",
    airports: [
      { name: "中部国際空港(セントレア)", code: "NGO", lines: "名鉄", tip: "名古屋市内から高単価。帰路は空回送注意。" },
    ],
    areas: [
      { id: "n1", name: "栄・錦",       grade: "S", peak: "金土深夜",    tips: "名古屋最大の歓楽街。0〜3時が最強。錦三（キンサン）周辺が核心。", lat: 35.1693, lng: 136.9091 },
      { id: "n2", name: "名古屋駅周辺", grade: "A", peak: "毎日夕・深夜", tips: "新幹線接続需要。太閤口（西口）側が穴場。", lat: 35.1706, lng: 136.8816 },
      { id: "n3", name: "大須・上前津", grade: "B", peak: "週末夜",       tips: "若者向け飲食・買い物帰りの需要。週末が中心。", lat: 35.1589, lng: 136.8999 },
    ],
    stations: [
      { name: "名古屋",   lines: "JR各線・新幹線・名鉄・近鉄・地下鉄各線", lastTrain: "約0:25", tip: "新幹線最終後の需要あり。桜通口・太閤口両方に需要。" },
      { name: "栄",       lines: "地下鉄東山線・名城線",                   lastTrain: "約0:20", tip: "繁華街直結。終電後の飲み客需要が集中。" },
    ],
    venueIds: ["other"],
  },

  // ========== 福岡県 ==========
  fukuoka: {
    id: "fukuoka", pref: "fukuoka",
    name: "福岡交通圏", short: "福岡",
    desc: "福岡市・北九州市・久留米市など",
    peakNote: "中洲の深夜需要が最強。天神は平日夕方の帰宅需要も安定。",
    taxiMeter: "初乗り500円（1.296km）以降264mごと100円（距離が伸びやすい）",
    restriction: "福岡県内は自由営業。県境越えは事前確認が必要。",
    airports: [
      { name: "福岡空港", code: "FUK", lines: "地下鉄空港線", tip: "市内から非常に近い。空港行き需要が高頻度で出る。" },
    ],
    areas: [
      { id: "f1", name: "中洲・川端",   grade: "S", peak: "毎日深夜",    tips: "九州最大の歓楽街。0〜3時が最強。ネオンが消えたあたりが拾い時。", lat: 33.6063, lng: 130.4091 },
      { id: "f2", name: "天神・大名",   grade: "S", peak: "平日夕・金土", tips: "百貨店・飲食街の帰宅需要。深夜も飲み帰り需要あり。", lat: 33.5904, lng: 130.3990 },
      { id: "f3", name: "博多駅周辺",   grade: "A", peak: "毎日夕・深夜", tips: "新幹線接続需要。博多口が主戦場。郊外行き長距離が出やすい。", lat: 33.5902, lng: 130.4208 },
      { id: "f4", name: "大橋・薬院",   grade: "B", peak: "平日夕",       tips: "住宅地帰宅需要。安定しているが単価は低め。", lat: 33.5704, lng: 130.4122 },
    ],
    stations: [
      { name: "博多",   lines: "JR各線・新幹線・地下鉄空港線", lastTrain: "約0:20", tip: "新幹線最終後の需要あり。博多口が主戦場。" },
      { name: "天神",   lines: "地下鉄空港線・七隈線・西鉄",   lastTrain: "約0:20", tip: "終電後の繁華街需要が集中する。" },
      { name: "中洲川端",lines: "地下鉄空港線・箱崎線",        lastTrain: "約0:20", tip: "中洲直結。終電後は独壇場になりやすい。" },
    ],
    venueIds: ["other"],
  },
};

// 会場マスタに圏情報を追加
const ZONE_VENUE_MAP = {
  tokubestu:      ["ariake_arena","bigsight","national_olympic","ajinomoto","budokan","marunouchi","toyosu_pit","zepp_haneda","other"],
  kita_tama:      ["ajinomoto","seibu_dome","other"],
  minami_tama:    ["other"],
  nishi_tama:     ["other"],
  yokohama:       ["pia_arena","nissan_stadium","yokohama_arena","other"],
  kanagawa_other: ["other"],
  chiba_city:     ["makuhari","zozomarineplus","other"],
  saitama:        ["saitama_super_arena","seibu_dome","other"],
  osaka_city:     ["kyocera_dome","intex_osaka","namba_city","other"],
  kyoto:          ["other"],
  kobe:           ["other"],
  nagoya:         ["other"],
  fukuoka:        ["other"],
};


// ===== DATA =====
const TOKYO_AREAS = [
  { id: 1, name: "新宿・歌舞伎町", grade: "S", peak: "金土深夜", tips: "歌舞伎町は0〜3時が最強。西新宿ビル街は18〜20時のビジネス需要が安定。", lat: 35.6938, lng: 139.7036, color: "#ef4444" },
  { id: 2, name: "渋谷・恵比寿", grade: "S", peak: "金土22時〜", tips: "渋谷はスクランブル周辺待機が鉄板。恵比寿は高単価が出やすい。", lat: 35.6580, lng: 139.7016, color: "#ef4444" },
  { id: 3, name: "六本木・赤坂", grade: "S", peak: "毎日深夜", tips: "六本木ヒルズ・ミッドタウン周辺は長距離が多い。外国人客も狙える。", lat: 35.6628, lng: 139.7315, color: "#ef4444" },
  { id: 4, name: "銀座・有楽町", grade: "A", peak: "平日18〜21時", tips: "クラブ帰り高単価あり。百貨店閉店後の客も多い。", lat: 35.6721, lng: 139.7649, color: "#f97316" },
  { id: 5, name: "品川・天王洲", grade: "A", peak: "平日朝夕", tips: "品川駅は新幹線需要。ビジネス客の羽田行きが稼ぎやすい。", lat: 35.6284, lng: 139.7387, color: "#f97316" },
  { id: 6, name: "池袋", grade: "A", peak: "金土深夜", tips: "東口・西口ともに深夜需要大。飲み客の郊外行き長距離が出る。", lat: 35.7295, lng: 139.7109, color: "#f97316" },
  { id: 7, name: "浅草・上野", grade: "B", peak: "昼間・休日", tips: "観光客多め。空港行きを狙う戦略が有効。", lat: 35.7147, lng: 139.7967, color: "#eab308" },
  { id: 8, name: "秋葉原・神田", grade: "B", peak: "平日昼・夕", tips: "ビジネス短距離多め。イベント時は跳ねる。", lat: 35.6984, lng: 139.7731, color: "#eab308" },
  { id: 9, name: "東京駅・丸の内", grade: "A", peak: "平日朝夕・深夜", tips: "新幹線・長距離バス連絡。高単価の郊外行き多い。", lat: 35.6812, lng: 139.7671, color: "#f97316" },
  { id: 10, name: "豊洲・有明", grade: "B", peak: "イベント時", tips: "ビッグサイト・豊洲PITのイベント時が勝負。平常時は薄い。", lat: 35.6453, lng: 139.7940, color: "#eab308" },
];

const AIRPORTS = [
  { name: "羽田空港", code: "HND", lines: "京急・モノレール", tip: "第3ターミナル国際線が深夜に狙い目。" },
  { name: "成田空港", code: "NRT", lines: "成田エクスプレス・スカイライナー", tip: "都心からは高単価だが遠い。帰路は空で走るリスクあり。" },
];

const MAJOR_STATIONS = [
  { name: "新宿", lines: "JR各線・小田急・京王・地下鉄", lastTrain: "約0:30", tip: "終電後のタクシー需要が最大。" },
  { name: "渋谷", lines: "JR・東横・田園都市・地下鉄", lastTrain: "約0:30", tip: "ハチ公口周辺が拾いやすい。" },
  { name: "池袋", lines: "JR各線・丸ノ内・有楽町・副都心", lastTrain: "約0:30", tip: "東西両口で客層が違う。" },
  { name: "品川", lines: "JR各線・京急", lastTrain: "約0:25", tip: "新幹線の最終接続客を狙う。" },
  { name: "東京", lines: "JR各線・地下鉄各線", lastTrain: "約0:30", tip: "丸の内口タクシー乗り場が穴場。" },
  { name: "六本木", lines: "日比谷線・大江戸線", lastTrain: "約0:30", tip: "大江戸線は0:14が最終。その後は需要急増。" },
];

// 会場マスタ（タクシー需要ランク・収容人数・攻略メモ付き）
const VENUES = [
  { id: "ariake_arena",     name: "有明アリーナ",           cap: 15000, taxiRank: 5, area: "有明・お台場",   tip: "終演後は一斉退場で争奪戦。りんかい線最終後は独壇場。" },
  { id: "bigsight",         name: "東京ビッグサイト",       cap: 90000, taxiRank: 5, area: "有明・お台場",   tip: "コミケ・展示会最終日終了後が最大。国際展示場駅側が拾いやすい。" },
  { id: "makuhari",         name: "幕張メッセ",             cap: 60000, taxiRank: 4, area: "幕張",           tip: "海浜幕張駅から遠い棟は特に需要大。帰路の高速代も込み単価高。" },
  { id: "national_olympic", name: "国立競技場",             cap: 68000, taxiRank: 4, area: "外苑前・信濃町", tip: "外苑前・信濃町・千駄ヶ谷各駅周辺が拾い場。渋滞注意。" },
  { id: "ajinomoto",        name: "味の素スタジアム",       cap: 50000, taxiRank: 4, area: "飛田給",         tip: "試合終了後は飛田給駅が大混雑。バス代替需要あり。" },
  { id: "budokan",          name: "日本武道館",             cap: 14000, taxiRank: 4, area: "九段下",         tip: "九段下・神保町方面が混雑。終演22時前後狙い。" },
  { id: "seibu_dome",       name: "ベルーナドーム",         cap: 33000, taxiRank: 3, area: "所沢",           tip: "西武球場前駅から混雑。池袋方面への長距離が出やすい。" },
  { id: "marunouchi",       name: "東京国際フォーラム",     cap:  5000, taxiRank: 3, area: "有楽町",         tip: "ビジネス・クラシック客中心。高単価の郊外行き多め。" },
  { id: "toyosu_pit",       name: "豊洲PIT",                cap:  3500, taxiRank: 3, area: "豊洲",           tip: "深夜終演が多い。豊洲駅終電後は独占状態になりやすい。" },
  { id: "pia_arena",        name: "Pアリーナ MM（横浜）",  cap: 10000, taxiRank: 3, area: "みなとみらい",   tip: "横浜エリアだが帰路需要で都内まで高単価あり。" },
  { id: "zepp_haneda",      name: "Zepp Haneda",            cap:  2716, taxiRank: 3, area: "羽田",           tip: "羽田空港近接。終演後に空港客と混在しやすい。" },
  { id: "other",            name: "その他・指定なし",        cap:     0, taxiRank: 1, area: "",               tip: "" },
  // 関東・関西・各都市会場
  { id: "nissan_stadium",    name: "日産スタジアム",         cap: 72000, taxiRank: 5, area: "新横浜",         tip: "試合・コンサート後は新横浜駅周辺が混雑。駅から離れた出口側が穴場。" },
  { id: "yokohama_arena",    name: "横浜アリーナ",           cap: 17000, taxiRank: 4, area: "新横浜",         tip: "新横浜駅直結。終演後の需要が出口付近に集中する。" },
  { id: "zozomarineplus",    name: "ZOZOマリンスタジアム",   cap: 30000, taxiRank: 4, area: "海浜幕張",       tip: "海浜幕張駅は終電が早い。試合終了後は独壇場になりやすい。" },
  { id: "saitama_super_arena",name:"さいたまスーパーアリーナ",cap:37000, taxiRank: 5, area: "さいたま新都心",  tip: "さいたま新都心駅直近。終演後は駅周辺に需要集中。浦和・大宮への中距離多め。" },
  { id: "kyocera_dome",      name: "京セラドーム大阪",       cap: 36000, taxiRank: 5, area: "ドーム前",       tip: "ドーム前駅周辺が主戦場。なんば・梅田方面への帰宅需要が多い。" },
  { id: "intex_osaka",       name: "インテックス大阪",       cap: 50000, taxiRank: 4, area: "住之江",         tip: "住之江は交通不便なためタクシー需要が高い。終了後は独壇場になりやすい。" },
  { id: "namba_city",        name: "なんばグランド花月周辺", cap:  2000, taxiRank: 3, area: "難波",           tip: "小規模だが難波の流し需要と合わせやすい。" },
];

// 初期イベントデータ（venue_idで会場と紐付け）
const INITIAL_EVENTS = [
  { id: 1, date: "2026-05-04", name: "コミックマーケット春",  venue_id: "bigsight",         type: "other",   note: "初日。国際展示場駅〜正面入口付近が狙い目", area: "" },
  { id: 2, date: "2026-05-04", name: "プロ野球 交流戦",       venue_id: "ajinomoto",        type: "sports",  note: "ナイター終了22時過ぎ。飛田給周辺で待機", area: "" },
  { id: 3, date: "2026-07-26", name: "隅田川花火大会",        venue_id: "other",            type: "festival",note: "終了後20〜23時が最大需要。混雑注意", area: "浅草・両国" },
  { id: 4, date: "2026-08-10", name: "夏フェス（仮）",        venue_id: "ariake_arena",     type: "concert", note: "深夜終演想定。りんかい線終電後に集中", area: "" },
];

const SHIFT_TYPES = [
  { id: "day", label: "日勤", start: "07:00", end: "19:00", hours: 12 },
  { id: "night", label: "夜勤", start: "19:00", end: "07:00", hours: 12 },
  { id: "alternating_day", label: "隔日（明け）", start: "07:00", end: "翌07:00", hours: 24 },
  { id: "custom", label: "カスタム", start: "", end: "", hours: 0 },
];

// ===== UTILS =====
const fmt = (n) => n?.toLocaleString("ja-JP") ?? "0";
const today = () => new Date().toISOString().split("T")[0];
const gradeColor = { S: "#ef4444", A: "#f97316", B: "#eab308", C: "#6b7280" };

// 締め日ロジック
// closeDay: 締め日（例: 15）
// 「N月分」= 前月(N-1)の(closeDay+1)日 ～ N月のcloseDay日
function getPeriodLabel(closeDay, date) {
  // date が属する「締め期間」のラベルを返す
  // 例: closeDay=15, date=2026-02-10 → "1月分"（1/16〜2/15）
  const d = new Date(date);
  const day = d.getDate();
  // この日がcloseDay以下なら「当月分」、超えていれば「翌月分」
  const billingMonth = day <= closeDay ? d.getMonth() + 1 : d.getMonth() + 2; // 1-indexed
  const billingYear  = day <= closeDay ? d.getFullYear()  : (d.getMonth() === 11 ? d.getFullYear() + 1 : d.getFullYear());
  return { year: billingYear, month: billingMonth, label: `${billingMonth}月分` };
}

function getPeriodRange(closeDay, billingYear, billingMonth) {
  // 期間の開始・終了日を返す
  // 開始: 前月のcloseDay+1
  const prevMonth = billingMonth === 1 ? 12 : billingMonth - 1;
  const prevYear  = billingMonth === 1 ? billingYear - 1 : billingYear;
  const startDay  = closeDay + 1;
  // 前月の日数チェック（startDayが前月の日数を超える場合は月末）
  const daysInPrev = new Date(prevYear, prevMonth, 0).getDate();
  const clampedStart = Math.min(startDay, daysInPrev);
  const start = `${prevYear}-${String(prevMonth).padStart(2,"0")}-${String(clampedStart).padStart(2,"0")}`;
  // 終了: 当月のcloseDay
  const daysInBilling = new Date(billingYear, billingMonth, 0).getDate();
  const clampedEnd = Math.min(closeDay, daysInBilling);
  const end   = `${billingYear}-${String(billingMonth).padStart(2,"0")}-${String(clampedEnd).padStart(2,"0")}`;
  return { start, end };
}

function groupRecordsByPeriod(records, closeDay) {
  // 全レコードを締め期間ごとにグループ化
  const groups = {};
  for (const r of records) {
    const { year, month } = getPeriodLabel(closeDay, r.date);
    const key = `${year}-${String(month).padStart(2,"0")}`;
    if (!groups[key]) groups[key] = { year, month, key, records: [] };
    groups[key].records.push(r);
  }
  // 新しい期間順にソート
  return Object.values(groups).sort((a, b) => b.key.localeCompare(a.key));
}

function getCurrentPeriodKey(closeDay) {
  const { year, month } = getPeriodLabel(closeDay, today());
  return `${year}-${String(month).padStart(2,"0")}`;
}


// ===== 交通情報マスタ =====
// Yahoo!路線情報の運行情報URL（路線コード付き）
const TRAIN_LINES = {
  tokubestu: [
    { name: "JR山手線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/22/0/", x: "https://x.com/JR_East_Info" },
    { name: "JR中央線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/25/0/", x: null },
    { name: "JR京浜東北線",   yahoo: "https://transit.yahoo.co.jp/traininfo/detail/23/0/", x: null },
    { name: "東京メトロ全線", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/51/0/", x: "https://x.com/tokyometro_info" },
    { name: "都営地下鉄全線", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/58/0/", x: null },
    { name: "小田急線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/35/0/", x: null },
    { name: "京王線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/36/0/", x: null },
    { name: "東急線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/42/0/", x: null },
    { name: "西武線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/38/0/", x: null },
    { name: "東武線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/39/0/", x: null },
    { name: "京急線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/41/0/", x: null },
  ],
  kita_tama: [
    { name: "JR中央線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/25/0/", x: null },
    { name: "JR南武線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/27/0/", x: null },
    { name: "京王線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/36/0/", x: null },
    { name: "西武線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/38/0/", x: null },
    { name: "多摩モノレール", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/92/0/", x: null },
  ],
  minami_tama: [
    { name: "JR横浜線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/28/0/", x: null },
    { name: "小田急線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/35/0/", x: null },
    { name: "京王線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/36/0/", x: null },
  ],
  nishi_tama: [
    { name: "JR青梅線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/26/0/", x: null },
    { name: "JR五日市線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/26/0/", x: null },
  ],
  yokohama: [
    { name: "JR東海道線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/24/0/", x: null },
    { name: "JR横浜線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/28/0/", x: null },
    { name: "東急東横線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/43/0/", x: null },
    { name: "東急田園都市線", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/44/0/", x: null },
    { name: "京急線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/41/0/", x: null },
    { name: "横浜市営地下鉄", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/68/0/", x: null },
    { name: "相鉄線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/48/0/", x: null },
  ],
  kanagawa_other: [
    { name: "JR東海道線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/24/0/", x: null },
    { name: "小田急線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/35/0/", x: null },
    { name: "江ノ電",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/74/0/", x: null },
  ],
  chiba_city: [
    { name: "JR総武線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/21/0/", x: null },
    { name: "JR京葉線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/30/0/", x: null },
    { name: "東武野田線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/40/0/", x: null },
    { name: "京成線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/45/0/", x: null },
  ],
  saitama: [
    { name: "JR京浜東北線",   yahoo: "https://transit.yahoo.co.jp/traininfo/detail/23/0/", x: null },
    { name: "JR高崎線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/32/0/", x: null },
    { name: "JR宇都宮線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/31/0/", x: null },
    { name: "東武東上線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/39/0/", x: null },
    { name: "西武新宿線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/38/0/", x: null },
    { name: "埼玉新都市交通", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/93/0/", x: null },
  ],
  osaka_city: [
    { name: "JR大阪環状線",   yahoo: "https://transit.yahoo.co.jp/traininfo/detail/113/0/", x: "https://x.com/JRWest_official" },
    { name: "JR東海道本線",   yahoo: "https://transit.yahoo.co.jp/traininfo/detail/111/0/", x: null },
    { name: "大阪メトロ全線", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/151/0/", x: "https://x.com/OsakaMetro_info" },
    { name: "阪急線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/131/0/", x: null },
    { name: "阪神線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/132/0/", x: null },
    { name: "近鉄線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/135/0/", x: null },
    { name: "南海線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/136/0/", x: null },
  ],
  kyoto: [
    { name: "JR琵琶湖線・京都線", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/111/0/", x: null },
    { name: "近鉄京都線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/135/0/", x: null },
    { name: "京都市営地下鉄", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/161/0/", x: null },
    { name: "阪急京都線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/131/0/", x: null },
  ],
  kobe: [
    { name: "JR神戸線",       yahoo: "https://transit.yahoo.co.jp/traininfo/detail/111/0/", x: null },
    { name: "阪急神戸線",     yahoo: "https://transit.yahoo.co.jp/traininfo/detail/131/0/", x: null },
    { name: "阪神線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/132/0/", x: null },
    { name: "神戸市営地下鉄", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/162/0/", x: null },
  ],
  nagoya: [
    { name: "JR東海道本線",   yahoo: "https://transit.yahoo.co.jp/traininfo/detail/210/0/", x: "https://x.com/JRCentral_info" },
    { name: "名鉄線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/231/0/", x: null },
    { name: "近鉄線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/135/0/", x: null },
    { name: "名古屋市営地下鉄",yahoo: "https://transit.yahoo.co.jp/traininfo/detail/251/0/", x: null },
  ],
  fukuoka: [
    { name: "JR鹿児島本線",   yahoo: "https://transit.yahoo.co.jp/traininfo/detail/311/0/", x: "https://x.com/jrkyushu_info" },
    { name: "西鉄線",         yahoo: "https://transit.yahoo.co.jp/traininfo/detail/331/0/", x: null },
    { name: "福岡市営地下鉄", yahoo: "https://transit.yahoo.co.jp/traininfo/detail/351/0/", x: null },
  ],
};

// 空港フライト情報URL
const AIRPORT_URLS = {
  HND: { name: "羽田空港", flightUrl: "https://tokyo-haneda.com/flight/flightInfo/dep.html", yahooUrl: "https://transit.yahoo.co.jp/airports/HND" },
  NRT: { name: "成田空港", flightUrl: "https://www.narita-airport.jp/jp/flight/", yahooUrl: "https://transit.yahoo.co.jp/airports/NRT" },
  KIX: { name: "関西空港", flightUrl: "https://www.kansai-airport.or.jp/flight/index.html", yahooUrl: "https://transit.yahoo.co.jp/airports/KIX" },
  ITM: { name: "伊丹空港", flightUrl: "https://www.osaka-airport.co.jp/flight/", yahooUrl: "https://transit.yahoo.co.jp/airports/ITM" },
  NGO: { name: "中部国際空港", flightUrl: "https://www.centrair.jp/flight/", yahooUrl: "https://transit.yahoo.co.jp/airports/NGO" },
  FUK: { name: "福岡空港", flightUrl: "https://www.fukuoka-airport.jp/flight/", yahooUrl: "https://transit.yahoo.co.jp/airports/FUK" },
};

// Yahoo!路線情報トップ（運行情報一覧）
const YAHOO_TRAIN_TOP = "https://transit.yahoo.co.jp/traininfo/area/4/";

// ===== COMPONENTS =====

function GradeTag({ grade }) {
  return (
    <span style={{
      background: gradeColor[grade] || "#6b7280",
      color: "#fff",
      fontWeight: 900,
      fontSize: 11,
      padding: "2px 7px",
      borderRadius: 4,
      letterSpacing: 1,
    }}>{grade}</span>
  );
}


// ===== インラインカレンダー =====
function InlineCalendar({ value, onChange, records }) {
  const now = new Date();
  const [viewYear, setViewYear] = useState(now.getFullYear());
  const [viewMonth, setViewMonth] = useState(now.getMonth()); // 0-indexed

  const firstDay = new Date(viewYear, viewMonth, 1).getDay();
  const daysInMonth = new Date(viewYear, viewMonth + 1, 0).getDate();
  const todayStr = now.toISOString().split("T")[0];

  // 記録済み日付セット
  const recordedDates = new Set(records.map(r => r.date));

  const prevMonth = () => {
    if (viewMonth === 0) { setViewYear(y => y - 1); setViewMonth(11); }
    else setViewMonth(m => m - 1);
  };
  const nextMonth = () => {
    if (viewMonth === 11) { setViewYear(y => y + 1); setViewMonth(0); }
    else setViewMonth(m => m + 1);
  };

  const monthName = `${viewYear}年${viewMonth + 1}月`;
  const DOW = ["日","月","火","水","木","金","土"];

  const cells = [];
  for (let i = 0; i < firstDay; i++) cells.push(null);
  for (let d = 1; d <= daysInMonth; d++) cells.push(d);

  return (
    <div style={{ background: "#0d0d1a", borderRadius: 10, padding: "12px 10px", border: "1px solid #2d2d4e" }}>
      {/* ヘッダー */}
      <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
        <button onClick={prevMonth} style={{ background: "none", border: "none", color: "#8b8bbd", fontSize: 18, cursor: "pointer", padding: "0 8px" }}>‹</button>
        <span style={{ color: "#e2e2ff", fontWeight: 700, fontSize: 14 }}>{monthName}</span>
        <button onClick={nextMonth} style={{ background: "none", border: "none", color: "#8b8bbd", fontSize: 18, cursor: "pointer", padding: "0 8px" }}>›</button>
      </div>
      {/* 曜日ヘッダー */}
      <div style={{ display: "grid", gridTemplateColumns: "repeat(7, 1fr)", marginBottom: 4 }}>
        {DOW.map((d, i) => (
          <div key={d} style={{ textAlign: "center", fontSize: 10, color: i === 0 ? "#ef4444" : i === 6 ? "#60a5fa" : "#6b6b9b", padding: "2px 0" }}>{d}</div>
        ))}
      </div>
      {/* 日付グリッド */}
      <div style={{ display: "grid", gridTemplateColumns: "repeat(7, 1fr)", gap: 2 }}>
        {cells.map((d, i) => {
          if (!d) return <div key={i} />;
          const dateStr = `${viewYear}-${String(viewMonth + 1).padStart(2, "0")}-${String(d).padStart(2, "0")}`;
          const isSelected = dateStr === value;
          const isToday = dateStr === todayStr;
          const isRecorded = recordedDates.has(dateStr);
          const dow = (firstDay + d - 1) % 7;
          const isSun = dow === 0, isSat = dow === 6;
          return (
            <button
              key={i}
              onClick={() => onChange(dateStr)}
              style={{
                background: isSelected ? "#7c3aed" : isToday ? "#1e1e3f" : "transparent",
                border: isToday && !isSelected ? "1px solid #4f46e5" : "1px solid transparent",
                borderRadius: 6, padding: "6px 2px", cursor: "pointer",
                display: "flex", flexDirection: "column", alignItems: "center", gap: 2,
              }}
            >
              <span style={{
                fontSize: 13, fontWeight: isSelected || isToday ? 700 : 400,
                color: isSelected ? "#fff" : isSun ? "#ef4444" : isSat ? "#60a5fa" : "#e2e2ff",
              }}>{d}</span>
              {isRecorded && <span style={{ width: 4, height: 4, borderRadius: "50%", background: isSelected ? "#fff" : "#c084fc", display: "block" }} />}
            </button>
          );
        })}
      </div>
      {/* 選択日表示 */}
      <div style={{ marginTop: 10, textAlign: "center", color: "#8b8bbd", fontSize: 12 }}>
        選択中: <span style={{ color: "#c084fc", fontWeight: 700 }}>{value}</span>
        {recordedDates.has(value) && <span style={{ color: "#f97316", marginLeft: 8, fontSize: 11 }}>⚠️ 記録済み</span>}
      </div>
    </div>
  );
}

// ===== 売上テンキー =====





// ===== 期間別履歴コンポーネント =====
function PeriodHistory({ allPeriods, closeDay, deleteRecord, curPeriodKey }) {
  const [openKey, setOpenKey] = useState(curPeriodKey);

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
      {allPeriods.map(period => {
        const isCurrent = period.key === curPeriodKey;
        const isOpen = openKey === period.key;
        const range = getPeriodRange(closeDay, period.year, period.month);
        const pTotal = period.records.reduce((s, r) => s + (r.sales || 0), 0);
        const pTrips = period.records.reduce((s, r) => s + (r.trips || 0), 0);

        return (
          <div key={period.key} style={{ background: "#1a1a2e", borderRadius: 12, border: isCurrent ? "1px solid #4f46e5" : "1px solid #2d2d4e", overflow: "hidden" }}>
            {/* ヘッダー（タップで開閉） */}
            <div
              onClick={() => setOpenKey(isOpen ? null : period.key)}
              style={{ padding: "12px 14px", cursor: "pointer", display: "flex", justifyContent: "space-between", alignItems: "center" }}
            >
              <div>
                <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                  <span style={{ color: isCurrent ? "#c084fc" : "#e2e2ff", fontWeight: 700, fontSize: 15 }}>
                    {period.month}月分
                  </span>
                  {isCurrent && <span style={{ background: "#4f46e5", color: "#fff", fontSize: 10, fontWeight: 700, padding: "1px 7px", borderRadius: 4 }}>今期</span>}
                </div>
                <div style={{ color: "#6b6b9b", fontSize: 11, marginTop: 2 }}>{range.start} 〜 {range.end}</div>
              </div>
              <div style={{ textAlign: "right" }}>
                <div style={{ color: "#c084fc", fontWeight: 700, fontSize: 16 }}>¥{fmt(pTotal)}</div>
                <div style={{ color: "#6b6b9b", fontSize: 11, marginTop: 1 }}>{period.records.length}日 · {fmt(pTrips)}件</div>
              </div>
            </div>

            {/* 明細（開いているときのみ） */}
            {isOpen && (
              <div style={{ borderTop: "1px solid #2d2d4e" }}>
                {period.records.sort((a,b) => b.date.localeCompare(a.date)).map(r => (
                  <div key={r.id} style={{ padding: "10px 14px", borderBottom: "1px solid #1a1a3e", display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                    <div>
                      <div style={{ color: "#8b8bbd", fontSize: 11 }}>{r.date} · {SHIFT_TYPES.find(s => s.id === r.shift)?.label || r.shift}</div>
                      <div style={{ color: "#c084fc", fontWeight: 700, fontSize: 16, marginTop: 2 }}>¥{fmt(r.sales)}</div>
                      <div style={{ color: "#6b6b9b", fontSize: 12, marginTop: 1 }}>
                        {r.trips ? `${r.trips}件` : ""}{r.distance ? ` · ${r.distance}km` : ""}{r.memo ? ` · ${r.memo}` : ""}
                      </div>
                    </div>
                    <button onClick={() => deleteRecord(r.id)} style={{ background: "none", border: "1px solid #3d3d6e", color: "#6b6b9b", borderRadius: 6, padding: "3px 8px", cursor: "pointer", fontSize: 11, flexShrink: 0 }}>削除</button>
                  </div>
                ))}
                {/* 期間集計フッター */}
                <div style={{ padding: "10px 14px", background: "#0d0d1a", display: "flex", justifyContent: "space-between" }}>
                  <span style={{ color: "#8b8bbd", fontSize: 12 }}>期間合計</span>
                  <span style={{ color: "#c084fc", fontWeight: 700, fontSize: 14 }}>¥{fmt(pTotal)}</span>
                </div>
              </div>
            )}
          </div>
        );
      })}
    </div>
  );
}

// ===== SALES TAB =====
function SalesTab({ records, setRecords, closeDay, defaultShift }) {
  const [form, setForm] = useState({
    date: today(), shift: defaultShift || "alternating_day", sales: "", trips: "", distance: "", memo: ""
  });
  const [goal, setGoal] = useState({});
  const [goalInput, setGoalInput] = useState("");
  const [workDaysInput, setWorkDaysInput] = useState("");
  const [showGoalEdit, setShowGoalEdit] = useState(false);

  useEffect(() => {
    storage.get("taxi_goal").then(v => {
      if (v) try {
        const g = JSON.parse(v);
        setGoal(g);
        setGoalInput(g.monthly || "");
        setWorkDaysInput(g.workDays || "");
      } catch {}
    });
  }, []);

  const saveGoal = () => {
    const g = { monthly: Number(goalInput), workDays: Number(workDaysInput) };
    setGoal(g);
    storage.set("taxi_goal", JSON.stringify(g));
    setShowGoalEdit(false);
  };

  const addRecord = () => {
    if (!form.sales) return;
    const newRec = { ...form, id: Date.now(), sales: Number(form.sales), trips: Number(form.trips), distance: Number(form.distance) };
    const updated = [newRec, ...records];
    setRecords(updated);
    storage.set("taxi_records", JSON.stringify(updated));
    setForm({ date: today(), shift: defaultShift || form.shift, sales: "", trips: "", distance: "", memo: "" });
  };

  const deleteRecord = (id) => {
    const updated = records.filter(r => r.id !== id);
    setRecords(updated);
    storage.set("taxi_records", JSON.stringify(updated));
  };

  // 締め日ベースの集計
  const curPeriodKey = getCurrentPeriodKey(closeDay);
  const { year: curYear, month: curMonth } = (() => { const [y,m] = curPeriodKey.split("-"); return { year: Number(y), month: Number(m) }; })();
  const curRange = getPeriodRange(closeDay, curYear, curMonth);
  const periodLabel = `${curMonth}月分`;
  const periodDesc  = `${curRange.start} 〜 ${curRange.end}`;

  const periodRecords = records.filter(r => r.date >= curRange.start && r.date <= curRange.end);
  const totalSales  = periodRecords.reduce((s, r) => s + (r.sales || 0), 0);
  const totalTrips  = periodRecords.reduce((s, r) => s + (r.trips || 0), 0);
  const workedDays  = periodRecords.length;
  const avgSales    = workedDays ? Math.round(totalSales / workedDays) : 0;
  const goalPct     = goal.monthly ? Math.min(Math.round(totalSales / goal.monthly * 100), 100) : null;

  const plannedWorkDays = goal.workDays || 0;
  const remainWorkDays  = Math.max(plannedWorkDays - workedDays, 0);
  const remaining       = goal.monthly ? Math.max(goal.monthly - totalSales, 0) : 0;
  const dailyTarget     = (remaining > 0 && remainWorkDays > 0) ? Math.ceil(remaining / remainWorkDays) : 0;
  const isOnTrack       = goal.monthly && workedDays > 0 ? (totalSales / workedDays) >= dailyTarget : null;

  // 過去の期間グループ（全履歴表示用）
  const allPeriods = groupRecordsByPeriod(records, closeDay);

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
      {/* 月次サマリー */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16, border: "1px solid #2d2d4e" }}>
        <div style={{ marginBottom: 10 }}>
          <div style={{ color: "#c084fc", fontWeight: 700, fontSize: 16 }}>{periodLabel}</div>
          <div style={{ color: "#6b6b9b", fontSize: 11, marginTop: 2 }}>{periodDesc}</div>
        </div>
        <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 10 }}>
          {[
            { label: "売上合計", value: `¥${fmt(totalSales)}`, big: true },
            { label: "出勤済み", value: `${workedDays}日${plannedWorkDays ? ` / ${plannedWorkDays}日` : ""}` },
            { label: "総乗車数", value: `${fmt(totalTrips)}件` },
            { label: "日平均売上", value: `¥${fmt(avgSales)}` },
          ].map(item => (
            <div key={item.label} style={{ background: "#0d0d1a", borderRadius: 8, padding: "10px 12px" }}>
              <div style={{ color: "#6b6b9b", fontSize: 11 }}>{item.label}</div>
              <div style={{ color: item.big ? "#c084fc" : "#e2e2ff", fontWeight: 700, fontSize: item.big ? 20 : 16, marginTop: 2 }}>{item.value}</div>
            </div>
          ))}
        </div>

        {/* 日ごと目標パネル */}
        {goal.monthly > 0 && plannedWorkDays > 0 && (
          <div style={{ marginTop: 12 }}>
            {remainWorkDays > 0 ? (
              <div style={{
                background: isOnTrack ? "#0a1f0a" : "#1f0a0a",
                border: `1px solid ${isOnTrack ? "#166534" : "#7f1d1d"}`,
                borderRadius: 10, padding: "12px 14px",
              }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                  <div>
                    <div style={{ color: "#8b8bbd", fontSize: 11, marginBottom: 4 }}>残り{remainWorkDays}日の1日あたり目標</div>
                    <div style={{ color: isOnTrack ? "#4ade80" : "#f87171", fontWeight: 900, fontSize: 26, letterSpacing: 1 }}>
                      ¥{fmt(dailyTarget)}
                    </div>
                    <div style={{ color: "#6b6b9b", fontSize: 11, marginTop: 4 }}>
                      残り ¥{fmt(remaining)} ÷ {remainWorkDays}日
                    </div>
                  </div>
                  <div style={{ textAlign: "right" }}>
                    <div style={{ fontSize: 20 }}>{isOnTrack ? "✅" : "⚠️"}</div>
                    <div style={{ color: isOnTrack ? "#4ade80" : "#f87171", fontSize: 11, fontWeight: 700, marginTop: 2 }}>
                      {isOnTrack ? "ペース良好" : "ペース不足"}
                    </div>
                    <div style={{ color: "#6b6b9b", fontSize: 10, marginTop: 2 }}>日平均 ¥{fmt(avgSales)}</div>
                  </div>
                </div>
              </div>
            ) : totalSales >= goal.monthly ? (
              <div style={{ background: "#0a1f0a", border: "1px solid #166534", borderRadius: 10, padding: "12px 14px", textAlign: "center" }}>
                <div style={{ fontSize: 22 }}>🎉</div>
                <div style={{ color: "#4ade80", fontWeight: 700, fontSize: 15, marginTop: 4 }}>月間目標達成！</div>
              </div>
            ) : (
              <div style={{ background: "#1f0a0a", border: "1px solid #7f1d1d", borderRadius: 10, padding: "12px 14px", textAlign: "center" }}>
                <div style={{ color: "#f87171", fontWeight: 700, fontSize: 13 }}>出勤日数消化済み · 残り ¥{fmt(remaining)}</div>
              </div>
            )}
          </div>
        )}

        {/* 目標進捗バー */}
        {goalPct !== null && (
          <div style={{ marginTop: 12 }}>
            <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, color: "#8b8bbd", marginBottom: 4 }}>
              <span>月間目標 ¥{fmt(goal.monthly)}</span>
              <span style={{ color: goalPct >= 100 ? "#4ade80" : "#c084fc", fontWeight: 700 }}>{goalPct}%</span>
            </div>
            <div style={{ background: "#0d0d1a", borderRadius: 4, height: 8, overflow: "hidden" }}>
              <div style={{ width: `${goalPct}%`, background: goalPct >= 100 ? "#4ade80" : "linear-gradient(90deg,#7c3aed,#c084fc)", height: "100%", borderRadius: 4, transition: "width 0.5s" }} />
            </div>
          </div>
        )}

        {/* 目標設定ボタン */}
        <button
          onClick={() => setShowGoalEdit(v => !v)}
          style={{ marginTop: 12, width: "100%", background: "#0d0d1a", border: "1px solid #2d2d4e", color: "#8b8bbd", borderRadius: 8, padding: "8px", fontSize: 12, cursor: "pointer" }}
        >
          {showGoalEdit ? "▲ 閉じる" : "⚙️ 目標・出勤日数を設定"}
        </button>

        {showGoalEdit && (
          <div style={{ marginTop: 10, display: "flex", flexDirection: "column", gap: 8 }}>
            <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 8 }}>
              <div>
                <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>月間目標（円）</div>
                <input type="number" placeholder="例: 600000" value={goalInput}
                  onChange={e => setGoalInput(e.target.value)} style={{ ...inputStyle, fontSize: 13 }} />
              </div>
              <div>
                <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>今月の出勤日数</div>
                <input type="number" placeholder="例: 16" value={workDaysInput}
                  onChange={e => setWorkDaysInput(e.target.value)} min={1} max={31}
                  style={{ ...inputStyle, fontSize: 13 }} />
              </div>
            </div>
            <button onClick={saveGoal} style={{ background: "#7c3aed", color: "#fff", border: "none", borderRadius: 8, padding: "9px", fontWeight: 700, cursor: "pointer", fontSize: 13 }}>
              保存する
            </button>
          </div>
        )}
      </div>

      {/* 入力フォーム */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>日報入力</div>
        <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>

          {/* カレンダー日付選択 */}
          <InlineCalendar value={form.date} onChange={d => setForm({ ...form, date: d })} records={records} />



          {/* 売上入力 */}
          <div>
            <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>売上金額 <span style={{ color: "#ef4444" }}>*</span></div>
            <input
              type="number"
              placeholder="例: 50000"
              value={form.sales}
              onChange={e => setForm({ ...form, sales: e.target.value })}
              style={{ ...inputStyle, fontSize: 16 }}
            />
          </div>

          {/* 乗車件数・走行距離 */}
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 8 }}>
            <div>
              <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>乗車件数</div>
              <input type="number" placeholder="例: 20" value={form.trips}
                onChange={e => setForm({ ...form, trips: e.target.value })}
                style={inputStyle} />
            </div>
            <div>
              <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>走行距離(km)</div>
              <input type="number" placeholder="例: 150" value={form.distance}
                onChange={e => setForm({ ...form, distance: e.target.value })}
                style={inputStyle} />
            </div>
          </div>

          {/* メモ */}
          <input type="text" placeholder="メモ（任意）" value={form.memo}
            onChange={e => setForm({ ...form, memo: e.target.value })} style={inputStyle} />

          <button onClick={addRecord} style={{ background: "linear-gradient(135deg,#7c3aed,#4f46e5)", color: "#fff", border: "none", borderRadius: 8, padding: "13px", fontWeight: 700, fontSize: 15, cursor: "pointer", letterSpacing: 1 }}>
            ＋ 記録する
          </button>
        </div>
      </div>

      {/* 期間別履歴 */}
      {allPeriods.length === 0 ? (
        <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 20, border: "1px solid #2d2d4e", textAlign: "center", color: "#4b4b7b", fontSize: 13 }}>
          記録がありません
        </div>
      ) : (
        <PeriodHistory allPeriods={allPeriods} closeDay={closeDay} deleteRecord={deleteRecord} curPeriodKey={curPeriodKey} />
      )}
    </div>
  );
}

// ===== GPS UTILS =====
const gradeScore = { S: 3, A: 2, B: 1, C: 0 };

// Haversine距離計算（km）
function calcDistance(lat1, lng1, lat2, lng2) {
  const R = 6371;
  const dLat = (lat2 - lat1) * Math.PI / 180;
  const dLng = (lng2 - lng1) * Math.PI / 180;
  const a = Math.sin(dLat / 2) ** 2 + Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) * Math.sin(dLng / 2) ** 2;
  return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
}

// 現在地から各エリアに「スコア＝ランク÷距離」で期待値計算
function calcExpectedValue(userLat, userLng, areas) {
  return areas.map(area => {
    const dist = calcDistance(userLat, userLng, area.lat, area.lng);
    const score = gradeScore[area.grade] || 0;
    // 距離コスト込み期待値：近いほど・ランク高いほど高スコア
    const ev = score / (0.5 + dist * 0.15);
    return { ...area, dist: Math.round(dist * 10) / 10, ev: Math.round(ev * 100) / 100 };
  }).sort((a, b) => b.ev - a.ev);
}

// 現在地のエリア判定（最近傍）
function detectCurrentArea(userLat, userLng, areas) {
  let nearest = null, minDist = Infinity;
  for (const area of areas) {
    const d = calcDistance(userLat, userLng, area.lat, area.lng);
    if (d < minDist) { minDist = d; nearest = area; }
  }
  return { area: nearest, dist: Math.round(minDist * 10) / 10 };
}

// ===== AREA TAB =====
function AreaTab({ zone }) {
  const [selected, setSelected] = useState(null);
  const [filter, setFilter] = useState("all");
  const [gpsState, setGpsState] = useState("idle"); // idle | loading | ok | error
  const [userPos, setUserPos] = useState(null);
  const [rankedAreas, setRankedAreas] = useState(null);
  const [currentArea, setCurrentArea] = useState(null);
  const [lastUpdated, setLastUpdated] = useState(null);

  const zoneAreas = ALL_ZONES[zone]?.areas || TOKYO_AREAS;
  const filtered = filter === "all" ? zoneAreas : zoneAreas.filter(a => a.grade === filter);

  const fetchGPS = () => {
    if (!navigator.geolocation) {
      setGpsState("error");
      return;
    }
    setGpsState("loading");
    navigator.geolocation.getCurrentPosition(
      (pos) => {
        const { latitude: lat, longitude: lng } = pos.coords;
        setUserPos({ lat, lng });
        const areas = ALL_ZONES[zone]?.areas || TOKYO_AREAS;
        const ranked = calcExpectedValue(lat, lng, areas);
        setRankedAreas(ranked);
        const { area, dist } = detectCurrentArea(lat, lng, areas);
        setCurrentArea({ area, dist });
        setLastUpdated(new Date());
        setGpsState("ok");
      },
      () => setGpsState("error"),
      { enableHighAccuracy: true, timeout: 10000 }
    );
  };

  const topArea = rankedAreas?.[0];
  const currentRank = rankedAreas ? rankedAreas.findIndex(a => a.id === currentArea?.area?.id) : -1;
  const shouldMove = currentRank > 0 && topArea && (topArea.ev - (rankedAreas?.[currentRank]?.ev || 0)) > 0.3;

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>

      {/* GPS・おすすめパネル */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
          <div style={{ color: "#8b8bbd", fontSize: 12, letterSpacing: 1 }}>📍 現在地から分析</div>
          <button
            onClick={fetchGPS}
            disabled={gpsState === "loading"}
            style={{
              background: gpsState === "ok" ? "#14532d" : "#7c3aed",
              color: "#fff", border: "none", borderRadius: 6,
              padding: "5px 14px", fontSize: 12, fontWeight: 700,
              cursor: gpsState === "loading" ? "not-allowed" : "pointer",
              opacity: gpsState === "loading" ? 0.7 : 1
            }}
          >
            {gpsState === "loading" ? "取得中…" : gpsState === "ok" ? "🔄 更新" : "📍 GPS取得"}
          </button>
        </div>

        {gpsState === "idle" && (
          <div style={{ color: "#4b4b7b", fontSize: 13, textAlign: "center", padding: "16px 0" }}>
            GPS取得ボタンを押すと<br />現在地から最適エリアを判定します
          </div>
        )}

        {gpsState === "error" && (
          <div style={{ background: "#2a1010", borderRadius: 8, padding: 12, color: "#f87171", fontSize: 13 }}>
            ⚠️ 位置情報の取得に失敗しました。<br />ブラウザの位置情報許可を確認してください。
          </div>
        )}

        {gpsState === "loading" && (
          <div style={{ color: "#8b8bbd", fontSize: 13, textAlign: "center", padding: "16px 0" }}>
            🛰️ 位置情報を取得しています…
          </div>
        )}

        {gpsState === "ok" && rankedAreas && currentArea && (
          <>
            {/* 現在地判定 */}
            <div style={{ background: "#0d0d1a", borderRadius: 10, padding: "10px 12px", marginBottom: 10 }}>
              <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>現在いるエリア（推定）</div>
              <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                <div>
                  <span style={{ color: "#e2e2ff", fontWeight: 700, fontSize: 15 }}>{currentArea.area.name}</span>
                  <span style={{ color: "#6b6b9b", fontSize: 12, marginLeft: 8 }}>中心まで約{currentArea.dist}km</span>
                </div>
                <GradeTag grade={currentArea.area.grade} />
              </div>
              {lastUpdated && (
                <div style={{ color: "#4b4b7b", fontSize: 11, marginTop: 4 }}>
                  更新: {lastUpdated.getHours()}:{String(lastUpdated.getMinutes()).padStart(2, "0")}
                </div>
              )}
            </div>

            {/* 移動推奨バナー or 現地維持バナー */}
            {shouldMove ? (
              <div style={{
                background: "linear-gradient(135deg, #3b0764, #1e0a3c)",
                borderRadius: 10, padding: "14px 14px",
                border: "1px solid #7c3aed",
                marginBottom: 10
              }}>
                <div style={{ color: "#f0abfc", fontWeight: 900, fontSize: 14, marginBottom: 6 }}>
                  🚀 移動推奨
                </div>
                <div style={{ color: "#e2e2ff", fontWeight: 700, fontSize: 17, marginBottom: 2 }}>
                  → {topArea.name}
                </div>
                <div style={{ display: "flex", gap: 8, alignItems: "center", marginBottom: 6 }}>
                  <GradeTag grade={topArea.grade} />
                  <span style={{ color: "#8b8bbd", fontSize: 12 }}>約{topArea.dist}km先</span>
                </div>
                <div style={{ color: "#c084fc", fontSize: 12, lineHeight: 1.6 }}>
                  💡 {topArea.tips}
                </div>
                <div style={{ color: "#6b6b9b", fontSize: 11, marginTop: 8 }}>
                  ※ ランク・距離・時間帯を総合した期待値スコアで判定
                </div>
              </div>
            ) : (
              <div style={{
                background: "#0a1a10",
                borderRadius: 10, padding: "12px 14px",
                border: "1px solid #166534",
                marginBottom: 10
              }}>
                <div style={{ color: "#4ade80", fontWeight: 700, fontSize: 14 }}>
                  ✅ 現在地で営業継続がおすすめ
                </div>
                <div style={{ color: "#86efac", fontSize: 12, marginTop: 4 }}>
                  近隣に有意に期待値の高いエリアはありません
                </div>
              </div>
            )}

            {/* 期待値ランキング TOP5 */}
            <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 6, letterSpacing: 1 }}>期待値ランキング（距離・ランク総合）</div>
            <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
              {rankedAreas.slice(0, 5).map((area, i) => {
                const isCurrent = area.id === currentArea?.area?.id;
                const isTop = i === 0;
                return (
                  <div key={area.id} style={{
                    background: isCurrent ? "#1a2a1a" : isTop ? "#1e0a3c" : "#0d0d1a",
                    borderRadius: 8, padding: "8px 12px",
                    border: isCurrent ? "1px solid #166534" : isTop ? "1px solid #7c3aed" : "1px solid transparent",
                    display: "flex", justifyContent: "space-between", alignItems: "center"
                  }}>
                    <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                      <span style={{ color: isTop ? "#c084fc" : "#4b4b7b", fontWeight: 900, fontSize: 13, minWidth: 20 }}>
                        {i + 1}
                      </span>
                      <div>
                        <span style={{ color: "#e2e2ff", fontSize: 13, fontWeight: isCurrent || isTop ? 700 : 400 }}>
                          {area.name}
                        </span>
                        {isCurrent && <span style={{ color: "#4ade80", fontSize: 10, marginLeft: 6 }}>← 現在地</span>}
                      </div>
                    </div>
                    <div style={{ textAlign: "right" }}>
                      <GradeTag grade={area.grade} />
                      <div style={{ color: "#6b6b9b", fontSize: 10, marginTop: 2 }}>{area.dist}km</div>
                    </div>
                  </div>
                );
              })}
            </div>
          </>
        )}
      </div>

      {/* エリア一覧（既存） */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>エリア別戦略ランク</div>
        <div style={{ display: "flex", gap: 6, marginBottom: 12, flexWrap: "wrap" }}>
          {["all", "S", "A", "B"].map(g => (
            <button key={g} onClick={() => setFilter(g)} style={{
              background: filter === g ? gradeColor[g] || "#7c3aed" : "#0d0d1a",
              color: "#fff", border: "1px solid #2d2d4e", borderRadius: 6,
              padding: "4px 12px", fontSize: 12, cursor: "pointer", fontWeight: filter === g ? 700 : 400
            }}>{g === "all" ? "全表示" : `${g}ランク`}</button>
          ))}
        </div>
        <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
          {filtered.map(area => {
            const ranked = rankedAreas?.find(r => r.id === area.id);
            return (
              <div key={area.id} onClick={() => setSelected(selected?.id === area.id ? null : area)}
                style={{ background: selected?.id === area.id ? "#1e1e3f" : "#0d0d1a", borderRadius: 10, padding: "12px 14px", cursor: "pointer", border: selected?.id === area.id ? "1px solid #7c3aed" : "1px solid transparent", transition: "all 0.2s" }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                  <span style={{ color: "#e2e2ff", fontWeight: 600, fontSize: 14 }}>{area.name}</span>
                  <div style={{ display: "flex", gap: 6, alignItems: "center" }}>
                    {ranked && <span style={{ color: "#6b6b9b", fontSize: 11 }}>{ranked.dist}km</span>}
                    <GradeTag grade={area.grade} />
                  </div>
                </div>
                <div style={{ color: "#8b8bbd", fontSize: 12, marginTop: 4 }}>ピーク: {area.peak}</div>
                {selected?.id === area.id && (
                  <div style={{ marginTop: 10, background: "#0d0d1a", borderRadius: 8, padding: "10px 12px", borderLeft: "3px solid #7c3aed" }}>
                    <div style={{ color: "#c084fc", fontSize: 12, lineHeight: 1.6 }}>💡 {area.tips}</div>
                  </div>
                )}
              </div>
            );
          })}
        </div>
      </div>

      {/* 凡例 */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 8, letterSpacing: 1 }}>ランク基準</div>
        {[
          { g: "S", desc: "深夜・週末に高確率で高売上。最優先エリア" },
          { g: "A", desc: "安定した需要あり。時間帯選択が重要" },
          { g: "B", desc: "特定条件下のみ有効。イベント確認必須" },
        ].map(item => (
          <div key={item.g} style={{ display: "flex", gap: 8, alignItems: "flex-start", marginBottom: 6 }}>
            <GradeTag grade={item.g} />
            <span style={{ color: "#8b8bbd", fontSize: 12, lineHeight: 1.5 }}>{item.desc}</span>
          </div>
        ))}
      </div>
    </div>
  );
}

// ===== EVENT TAB =====
const typeIcon = { festival: "🎆", sports: "🏃", concert: "🎵", other: "📌" };
const typeLabel = { festival: "花火・祭り", sports: "スポーツ", concert: "ライブ・コンサート", other: "その他" };
const taxiRankLabel = { 5: "需要◎", 4: "需要○", 3: "需要△", 2: "需要▲", 1: "需要－" };
const taxiRankColor = { 5: "#ef4444", 4: "#f97316", 3: "#eab308", 2: "#6b7280", 1: "#4b4b7b" };

function EventTab({ zone }) {
  const [events, setEvents] = useState(INITIAL_EVENTS);
  useEffect(() => { storage.get("taxi_events").then(v => { if (v) try { setEvents(JSON.parse(v)); } catch {} }); }, []);
  const [form, setForm] = useState({ date: today(), name: "", area: "", venue_id: "other", type: "festival", note: "" });
  const [showForm, setShowForm] = useState(false);

  const addEvent = () => {
    if (!form.name || !form.date) return;
    const updated = [...events, { ...form, id: Date.now() }];
    setEvents(updated);
    storage.set("taxi_events", JSON.stringify(updated));
    setForm({ date: today(), name: "", area: "", venue_id: "other", type: "festival", note: "" });
    setShowForm(false);
  };

  const deleteEvent = (id) => {
    const updated = events.filter(e => e.id !== id);
    setEvents(updated);
    storage.set("taxi_events", JSON.stringify(updated));
  };

  // 当日のみ絞り込み → タクシー需要ランク降順 → 収容人数降順でソート
  const todayStr = today();
  const zoneVenueIds = ZONE_VENUE_MAP[zone] || ["other"];
  const todayEvents = events
    .filter(e => e.date === todayStr && (zoneVenueIds.includes(e.venue_id) || e.venue_id === "other"))
    .map(ev => {
      const venue = VENUES.find(v => v.id === ev.venue_id) || VENUES.find(v => v.id === "other");
      return { ...ev, venue };
    })
    .sort((a, b) => {
      const rankDiff = (b.venue?.taxiRank || 0) - (a.venue?.taxiRank || 0);
      if (rankDiff !== 0) return rankDiff;
      return (b.venue?.cap || 0) - (a.venue?.cap || 0);
    });

  // 直近7日（明日〜7日後）のプレビュー
  const soon = events
    .filter(e => e.date > todayStr && e.date <= new Date(Date.now() + 7*86400000).toISOString().split("T")[0]
      && (zoneVenueIds.includes(e.venue_id) || e.venue_id === "other"))
    .map(ev => {
      const venue = VENUES.find(v => v.id === ev.venue_id) || VENUES.find(v => v.id === "other");
      return { ...ev, venue };
    })
    .sort((a, b) => {
      if (a.date !== b.date) return a.date.localeCompare(b.date);
      return (b.venue?.taxiRank || 0) - (a.venue?.taxiRank || 0);
    });

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>

      {/* 本日のイベント */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
          <div>
            <div style={{ color: "#8b8bbd", fontSize: 12, letterSpacing: 1 }}>📅 本日のイベント</div>
            <div style={{ color: "#4b4b7b", fontSize: 11, marginTop: 2 }}>需要ランク順に表示</div>
          </div>
          <button onClick={() => setShowForm(!showForm)} style={{ background: "#7c3aed", color: "#fff", border: "none", borderRadius: 6, padding: "5px 14px", fontSize: 12, cursor: "pointer", fontWeight: 700 }}>
            {showForm ? "閉じる" : "＋ 追加"}
          </button>
        </div>

        {/* 追加フォーム */}
        {showForm && (
          <div style={{ marginBottom: 12, background: "#0d0d1a", borderRadius: 10, padding: 12, display: "flex", flexDirection: "column", gap: 8 }}>
            <input type="date" value={form.date} onChange={e => setForm({ ...form, date: e.target.value })} style={inputStyle} />
            <input type="text" placeholder="イベント名*" value={form.name} onChange={e => setForm({ ...form, name: e.target.value })} style={inputStyle} />
            <select value={form.venue_id} onChange={e => setForm({ ...form, venue_id: e.target.value })} style={inputStyle}>
              {VENUES.map(v => (
                <option key={v.id} value={v.id}>
                  {v.name}{v.cap > 0 ? `（〜${v.cap.toLocaleString()}人）` : ""}
                </option>
              ))}
            </select>
            <input type="text" placeholder="エリア補足（任意）" value={form.area} onChange={e => setForm({ ...form, area: e.target.value })} style={inputStyle} />
            <select value={form.type} onChange={e => setForm({ ...form, type: e.target.value })} style={inputStyle}>
              <option value="festival">花火・祭り</option>
              <option value="sports">スポーツ</option>
              <option value="concert">ライブ・コンサート</option>
              <option value="other">その他</option>
            </select>
            <input type="text" placeholder="作戦メモ（任意）" value={form.note} onChange={e => setForm({ ...form, note: e.target.value })} style={inputStyle} />
            <button onClick={addEvent} style={{ background: "#7c3aed", color: "#fff", border: "none", borderRadius: 8, padding: 10, fontWeight: 700, cursor: "pointer" }}>登録する</button>
          </div>
        )}

        {/* 本日イベント一覧 */}
        {todayEvents.length === 0 ? (
          <div style={{ background: "#0d0d1a", borderRadius: 10, padding: "20px 0", textAlign: "center" }}>
            <div style={{ color: "#4b4b7b", fontSize: 13 }}>本日の登録イベントなし</div>
            <div style={{ color: "#3b3b5b", fontSize: 11, marginTop: 4 }}>＋追加から登録できます</div>
          </div>
        ) : (
          <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
            {todayEvents.map((ev, i) => {
              const venue = ev.venue;
              const areaStr = ev.area || venue?.area || "";
              const isTop = i === 0;
              return (
                <div key={ev.id} style={{
                  background: isTop ? "linear-gradient(135deg, #1e0a3c, #0d0d1a)" : "#0d0d1a",
                  borderRadius: 12, padding: "12px 14px",
                  border: isTop ? "1px solid #7c3aed" : "1px solid #1a1a3e",
                  position: "relative", overflow: "hidden"
                }}>
                  {/* 順位バッジ */}
                  <div style={{ position: "absolute", top: 10, right: 12, display: "flex", flexDirection: "column", alignItems: "flex-end", gap: 4 }}>
                    <span style={{
                      background: taxiRankColor[venue?.taxiRank || 1],
                      color: "#fff", fontSize: 11, fontWeight: 700,
                      padding: "2px 8px", borderRadius: 4
                    }}>{taxiRankLabel[venue?.taxiRank || 1]}</span>
                    <button onClick={() => deleteEvent(ev.id)} style={{ background: "none", border: "1px solid #3d3d6e", color: "#6b6b9b", borderRadius: 4, padding: "2px 6px", cursor: "pointer", fontSize: 10 }}>削除</button>
                  </div>

                  {/* 順位マーク */}
                  {isTop && (
                    <div style={{ color: "#f0abfc", fontSize: 10, fontWeight: 700, marginBottom: 4, letterSpacing: 1 }}>
                      🔥 本日の最優先
                    </div>
                  )}

                  {/* イベント名 */}
                  <div style={{ color: "#e2e2ff", fontWeight: 700, fontSize: 15, paddingRight: 70, marginBottom: 4 }}>
                    {typeIcon[ev.type] || "📌"} {ev.name}
                  </div>

                  {/* 会場 */}
                  {venue && venue.id !== "other" && (
                    <div style={{ display: "flex", alignItems: "center", gap: 6, marginBottom: 4 }}>
                      <span style={{ color: "#c084fc", fontSize: 12, fontWeight: 600 }}>🏟 {venue.name}</span>
                      {venue.cap > 0 && (
                        <span style={{ color: "#6b6b9b", fontSize: 11 }}>最大{venue.cap.toLocaleString()}人</span>
                      )}
                    </div>
                  )}

                  {/* エリア */}
                  {areaStr && (
                    <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 4 }}>📍 {areaStr}</div>
                  )}

                  {/* 会場の攻略tip */}
                  {venue?.tip && venue.id !== "other" && (
                    <div style={{ color: "#a78bfa", fontSize: 12, lineHeight: 1.6, marginBottom: ev.note ? 6 : 0 }}>
                      🏟 {venue.tip}
                    </div>
                  )}

                  {/* 追加メモ */}
                  {ev.note && (
                    <div style={{ color: "#c084fc", fontSize: 12, lineHeight: 1.6 }}>
                      💡 {ev.note}
                    </div>
                  )}
                </div>
              );
            })}
          </div>
        )}
      </div>

      {/* 今後7日のプレビュー */}
      {soon.length > 0 && (
        <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
          <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>📆 直近7日のイベント</div>
          <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
            {soon.map(ev => {
              const venue = ev.venue;
              const daysLeft = Math.ceil((new Date(ev.date) - new Date(todayStr)) / 86400000);
              return (
                <div key={ev.id} style={{ background: "#0d0d1a", borderRadius: 10, padding: "10px 12px", display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                  <div style={{ flex: 1 }}>
                    <div style={{ color: "#e2e2ff", fontSize: 13, fontWeight: 600 }}>{typeIcon[ev.type] || "📌"} {ev.name}</div>
                    <div style={{ color: "#8b8bbd", fontSize: 11, marginTop: 2 }}>
                      {ev.date}（あと{daysLeft}日）
                      {venue && venue.id !== "other" ? ` · ${venue.name}` : ev.area ? ` · ${ev.area}` : ""}
                    </div>
                  </div>
                  <span style={{ color: taxiRankColor[venue?.taxiRank || 1], fontSize: 11, fontWeight: 700, whiteSpace: "nowrap", marginLeft: 8, marginTop: 2 }}>
                    {taxiRankLabel[venue?.taxiRank || 1]}
                  </span>
                </div>
              );
            })}
          </div>
        </div>
      )}

      {/* 会場リファレンス */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>🏟 主要会場 需要ランク一覧</div>
        <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
          {VENUES.filter(v => v.id !== "other" && (ZONE_VENUE_MAP[zone] || []).includes(v.id)).map(v => (
            <div key={v.id} style={{ background: "#0d0d1a", borderRadius: 8, padding: "8px 12px", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
              <div>
                <span style={{ color: "#e2e2ff", fontSize: 13, fontWeight: 600 }}>{v.name}</span>
                <span style={{ color: "#6b6b9b", fontSize: 11, marginLeft: 8 }}>{v.area}</span>
              </div>
              <span style={{ color: taxiRankColor[v.taxiRank], fontSize: 12, fontWeight: 700 }}>{taxiRankLabel[v.taxiRank]}</span>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}



// ===== 締め日設定コンポーネント =====
function CloseDaySetting({ closeDay, setCloseDay }) {
  const [input, setInput] = useState(String(closeDay));
  const [saved, setSaved] = useState(false);

  const presets = [10, 15, 20, 25, 28];

  const save = (day) => {
    const n = Number(day);
    if (n < 1 || n > 28) return;
    setCloseDay(n);
    setInput(String(n));
    storage.set("taxi_close_day", String(n));
    setSaved(true);
    setTimeout(() => setSaved(false), 1500);
  };

  const exRange = getPeriodRange(closeDay, new Date().getFullYear(), new Date().getMonth() + 1 <= closeDay ? new Date().getMonth() + 1 : new Date().getMonth() + 2);

  return (
    <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16, border: "1px solid #2d2d4e" }}>
      <div style={{ color: "#c084fc", fontWeight: 700, fontSize: 14, marginBottom: 4 }}>売上締め日の設定</div>
      <div style={{ color: "#6b6b9b", fontSize: 12, marginBottom: 12, lineHeight: 1.6 }}>
        締め日を設定すると売上が「N月分」単位で集計されます。
      </div>

      {/* プリセット */}
      <div style={{ display: "flex", gap: 8, marginBottom: 12, flexWrap: "wrap" }}>
        {presets.map(d => (
          <button key={d} onClick={() => save(d)} style={{
            background: closeDay === d ? "#7c3aed" : "#0d0d1a",
            border: closeDay === d ? "1px solid #a78bfa" : "1px solid #2d2d4e",
            color: closeDay === d ? "#fff" : "#8b8bbd",
            borderRadius: 8, padding: "8px 14px", cursor: "pointer", fontWeight: closeDay === d ? 700 : 400, fontSize: 14,
          }}>{d}日</button>
        ))}
      </div>

      {/* カスタム入力 */}
      <div style={{ display: "flex", gap: 8 }}>
        <input
          type="number" min={1} max={28}
          value={input}
          onChange={e => setInput(e.target.value)}
          placeholder="1〜28"
          style={{ ...inputStyle, flex: 1 }}
        />
        <button onClick={() => save(input)} style={{ background: "#7c3aed", color: "#fff", border: "none", borderRadius: 8, padding: "0 16px", fontWeight: 700, cursor: "pointer", fontSize: 13, flexShrink: 0 }}>
          設定
        </button>
      </div>

      {/* 現在の締め日と期間プレビュー */}
      <div style={{ marginTop: 12, background: "#0d0d1a", borderRadius: 8, padding: "10px 12px", borderLeft: "3px solid #7c3aed" }}>
        <div style={{ color: "#8b8bbd", fontSize: 11, marginBottom: 4 }}>現在の設定</div>
        <div style={{ color: "#e2e2ff", fontWeight: 700, fontSize: 14 }}>毎月{closeDay}日締め</div>
        <div style={{ color: "#6b6b9b", fontSize: 12, marginTop: 4 }}>
          今期: {exRange.start} 〜 {exRange.end}
        </div>
      </div>

      {saved && (
        <div style={{ marginTop: 10, background: "#0a2a10", border: "1px solid #166534", borderRadius: 8, padding: "8px 12px", color: "#4ade80", fontSize: 13, fontWeight: 700, textAlign: "center" }}>
          ✅ 締め日を保存しました
        </div>
      )}
    </div>
  );
}


// ===== 勤務体系設定コンポーネント =====
function ShiftSetting({ defaultShift, setDefaultShift }) {
  const save = (shiftId) => {
    setDefaultShift(shiftId);
    storage.set("taxi_default_shift", shiftId);
  };
  return (
    <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16, border: "1px solid #2d2d4e" }}>
      <div style={{ color: "#c084fc", fontWeight: 700, fontSize: 14, marginBottom: 4 }}>勤務体系の設定</div>
      <div style={{ color: "#6b6b9b", fontSize: 12, marginBottom: 12, lineHeight: 1.6 }}>
        日報入力時のデフォルト勤務体系を選択します。
      </div>
      <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
        {SHIFT_TYPES.map(s => {
          const isSelected = defaultShift === s.id;
          return (
            <button key={s.id} onClick={() => save(s.id)} style={{
              background: isSelected ? "#1e1e3f" : "#0d0d1a",
              border: isSelected ? "1px solid #7c3aed" : "1px solid #2d2d4e",
              borderRadius: 10, padding: "12px 14px", cursor: "pointer",
              display: "flex", justifyContent: "space-between", alignItems: "center",
            }}>
              <div style={{ textAlign: "left" }}>
                <div style={{ color: isSelected ? "#e2e2ff" : "#a0a0c0", fontWeight: isSelected ? 700 : 400, fontSize: 14 }}>
                  {isSelected ? "✅ " : ""}{s.label}
                </div>
                {s.start && (
                  <div style={{ color: "#6b6b9b", fontSize: 12, marginTop: 2 }}>{s.start}〜{s.end}</div>
                )}
              </div>
              {isSelected && (
                <span style={{ background: "#7c3aed", color: "#fff", fontSize: 10, fontWeight: 700, padding: "2px 8px", borderRadius: 4 }}>設定中</span>
              )}
            </button>
          );
        })}
      </div>
    </div>
  );
}

// ===== SETTINGS TAB =====
function SettingsTab({ zone, setZone, closeDay, setCloseDay, defaultShift, setDefaultShift }) {
  const [saved, setSaved] = useState(false);
  const [openPref, setOpenPref] = useState(() => {
    return ALL_ZONES[zone]?.pref || "tokyo";
  });

  const handleSave = (zoneId) => {
    setZone(zoneId);
    storage.set("taxi_zone", zoneId);
    setSaved(true);
    setTimeout(() => setSaved(false), 2000);
  };

  const prefColor = {
    tokyo: "#7c3aed", kanagawa: "#0e7490", chiba: "#0f766e",
    saitama: "#b45309", osaka: "#be123c", kyoto: "#9a3412",
    hyogo: "#1e40af", aichi: "#15803d", fukuoka: "#a16207",
  };

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 14 }}>
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#c084fc", fontWeight: 700, fontSize: 14, marginBottom: 4 }}>営業交通圏の設定</div>
        <div style={{ color: "#6b6b9b", fontSize: 12, marginBottom: 14, lineHeight: 1.6 }}>
          選択した交通圏に合わせてエリア・イベント・交通情報が最適化されます。
        </div>

        {/* 都道府県タブ */}
        <div style={{ display: "flex", flexWrap: "wrap", gap: 6, marginBottom: 14 }}>
          {PREFECTURES.map(p => (
            <button key={p.id} onClick={() => setOpenPref(p.id)} style={{
              background: openPref === p.id ? (prefColor[p.id] || "#7c3aed") : "#0d0d1a",
              color: "#fff", border: "none", borderRadius: 6,
              padding: "5px 12px", fontSize: 12, fontWeight: openPref === p.id ? 700 : 400,
              cursor: "pointer",
            }}>{p.name}</button>
          ))}
        </div>

        {/* 選択都道府県の交通圏一覧 */}
        {Object.values(ALL_ZONES)
          .filter(z => z.pref === openPref)
          .map(z => {
            const isSelected = zone === z.id;
            const pc = prefColor[z.pref] || "#7c3aed";
            return (
              <div key={z.id} onClick={() => handleSave(z.id)} style={{
                background: isSelected ? "#1a1240" : "#0d0d1a",
                border: isSelected ? `1px solid ${pc}` : "1px solid #2d2d4e",
                borderRadius: 12, padding: "14px 16px", marginBottom: 10,
                cursor: "pointer", transition: "all 0.2s",
              }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 4 }}>
                  <div style={{ color: isSelected ? "#e2e2ff" : "#a0a0c0", fontWeight: 700, fontSize: 15 }}>
                    {isSelected ? "✅ " : ""}{z.name}
                  </div>
                  {isSelected && (
                    <span style={{ background: pc, color: "#fff", fontSize: 10, fontWeight: 700, padding: "2px 8px", borderRadius: 4 }}>設定中</span>
                  )}
                </div>
                <div style={{ color: "#6b6b9b", fontSize: 12, marginBottom: 6 }}>{z.desc}</div>
                <div style={{ color: isSelected ? "#c084fc" : "#4b4b7b", fontSize: 11, lineHeight: 1.6 }}>
                  💡 {z.peakNote}
                </div>
                {isSelected && (
                  <div style={{ marginTop: 10, background: "#090920", borderRadius: 8, padding: "8px 12px", borderLeft: `3px solid ${pc}` }}>
                    <div style={{ color: "#8b8bbd", fontSize: 11, marginBottom: 2 }}>メーター料金</div>
                    <div style={{ color: "#e2e2ff", fontSize: 12 }}>{z.taxiMeter}</div>
                    <div style={{ color: "#8b8bbd", fontSize: 11, marginTop: 6, marginBottom: 2 }}>乗り入れ規制</div>
                    <div style={{ color: "#e2e2ff", fontSize: 12 }}>{z.restriction}</div>
                  </div>
                )}
              </div>
            );
          })}

        {saved && (
          <div style={{ background: "#0a2a10", border: "1px solid #166534", borderRadius: 8, padding: "10px 14px", color: "#4ade80", fontSize: 13, fontWeight: 700, textAlign: "center" }}>
            ✅ 設定を保存しました
          </div>
        )}
      </div>

      {/* 勤務体系設定 */}
      <ShiftSetting defaultShift={defaultShift} setDefaultShift={setDefaultShift} />

      {/* 締め日設定 */}
      <CloseDaySetting closeDay={closeDay} setCloseDay={setCloseDay} />

      {/* アプリ情報 */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>アプリ情報</div>
        <div style={{ color: "#6b6b9b", fontSize: 12, lineHeight: 1.8 }}>
          <div>🚖 TAXI MANAGER</div>
          <div>対応エリア：東京・神奈川・千葉・埼玉・大阪・京都・兵庫・愛知・福岡</div>
          <div>データはすべてブラウザ内に保存されます</div>
        </div>
      </div>
    </div>
  );
}

// ===== TRANSPORT TAB =====
function TransportTab({ zone }) {
  const [now, setNow] = useState(new Date());
  const [activeTab, setActiveTab] = useState("train"); // "train" | "airport"
  useEffect(() => { const t = setInterval(() => setNow(new Date()), 60000); return () => clearInterval(t); }, []);
  const h = now.getHours(), m = now.getMinutes();
  const timeStr = `${String(h).padStart(2, "0")}:${String(m).padStart(2, "0")}`;
  const isLateNight = h >= 23 || h < 5;
  const isRush = (h >= 7 && h < 10) || (h >= 17 && h < 21);

  const lines = TRAIN_LINES[zone] || TRAIN_LINES["tokubestu"];
  const airports = ALL_ZONES[zone]?.airports || AIRPORTS;
  const stations = ALL_ZONES[zone]?.stations || MAJOR_STATIONS;

  const openUrl = (url) => { window.open(url, "_blank", "noopener,noreferrer"); };

  // Yahoo!路線情報の運行情報エリアURL（関東/関西で分ける）
  const yahooAreaUrl = ["osaka_city","kyoto","kobe"].includes(zone)
    ? "https://transit.yahoo.co.jp/traininfo/area/6/"
    : ["nagoya"].includes(zone)
    ? "https://transit.yahoo.co.jp/traininfo/area/5/"
    : ["fukuoka"].includes(zone)
    ? "https://transit.yahoo.co.jp/traininfo/area/9/"
    : "https://transit.yahoo.co.jp/traininfo/area/4/";

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>

      {/* 現在時刻バナー */}
      <div style={{ background: isLateNight ? "#1a0a2e" : isRush ? "#1a1800" : "#1a1a2e", borderRadius: 12, padding: "12px 16px", border: `1px solid ${isLateNight ? "#7c3aed" : isRush ? "#d97706" : "#2d2d4e"}`, display: "flex", justifyContent: "space-between", alignItems: "center" }}>
        <div>
          <div style={{ color: "#e2e2ff", fontSize: 28, fontWeight: 900, letterSpacing: 2 }}>{timeStr}</div>
          <div style={{ fontSize: 13, fontWeight: 700, color: isLateNight ? "#c084fc" : isRush ? "#fbbf24" : "#6b8bbd", marginTop: 4 }}>
            {isLateNight ? "🌙 深夜帯" : isRush ? "⚡ ラッシュ帯" : "☀️ 通常時間帯"}
          </div>
        </div>
        {/* Yahoo!運行情報一括ボタン */}
        <button
          onClick={() => openUrl(yahooAreaUrl)}
          style={{ background: "#7c3aed", color: "#fff", border: "none", borderRadius: 10, padding: "10px 14px", cursor: "pointer", textAlign: "center", flexShrink: 0 }}
        >
          <div style={{ fontSize: 18 }}>🚨</div>
          <div style={{ fontSize: 11, fontWeight: 700, marginTop: 2 }}>運行情報</div>
          <div style={{ fontSize: 10, opacity: 0.8 }}>Yahoo!</div>
        </button>
      </div>

      {/* 電車 / 空港 タブ切替 */}
      <div style={{ display: "flex", background: "#0d0d1a", borderRadius: 10, padding: 4, gap: 4 }}>
        {[{ id: "train", label: "🚃 電車" }, { id: "airport", label: "✈️ 空港" }].map(t => (
          <button key={t.id} onClick={() => setActiveTab(t.id)} style={{
            flex: 1, background: activeTab === t.id ? "#1a1a2e" : "transparent",
            border: activeTab === t.id ? "1px solid #2d2d4e" : "1px solid transparent",
            color: activeTab === t.id ? "#e2e2ff" : "#6b6b9b",
            borderRadius: 8, padding: "10px 0", cursor: "pointer",
            fontWeight: activeTab === t.id ? 700 : 400, fontSize: 14,
          }}>{t.label}</button>
        ))}
      </div>

      {/* ===== 電車タブ ===== */}
      {activeTab === "train" && (
        <>
          {/* 路線別 遅延確認ボタン */}
          <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 10 }}>
              <div style={{ color: "#8b8bbd", fontSize: 12, letterSpacing: 1 }}>路線別 運行情報</div>
              <button onClick={() => openUrl(yahooAreaUrl)} style={{ background: "#4f46e5", color: "#fff", border: "none", borderRadius: 6, padding: "4px 10px", fontSize: 11, fontWeight: 700, cursor: "pointer" }}>
                一括確認 →
              </button>
            </div>
            <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
              {lines.map(line => (
                <div key={line.name} style={{ background: "#0d0d1a", borderRadius: 8, padding: "10px 12px", display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                  <span style={{ color: "#e2e2ff", fontSize: 14, fontWeight: 600 }}>{line.name}</span>
                  <div style={{ display: "flex", gap: 6 }}>
                    <button
                      onClick={() => openUrl(line.yahoo)}
                      style={{ background: "#7c3aed", color: "#fff", border: "none", borderRadius: 6, padding: "5px 12px", fontSize: 12, fontWeight: 700, cursor: "pointer" }}
                    >
                      遅延確認
                    </button>
                  </div>
                </div>
              ))}
            </div>
            <div style={{ color: "#4b4b7b", fontSize: 11, marginTop: 10, textAlign: "center" }}>
              タップするとYahoo!路線情報が開きます
            </div>
          </div>

          {/* 主要駅 終電・攻略 */}
          <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
            <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>主要駅 終電目安・攻略</div>
            <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
              {stations.map(st => (
                <div key={st.name} style={{ background: "#0d0d1a", borderRadius: 10, padding: "10px 14px" }}>
                  <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                    <span style={{ color: "#e2e2ff", fontWeight: 700, fontSize: 14 }}>{st.name}</span>
                    <span style={{ color: "#ef4444", fontSize: 12, fontWeight: 700 }}>終電〜{st.lastTrain}</span>
                  </div>
                  <div style={{ color: "#8b8bbd", fontSize: 11, marginTop: 2 }}>{st.lines}</div>
                  <div style={{ color: "#c084fc", fontSize: 12, marginTop: 6 }}>💡 {st.tip}</div>
                </div>
              ))}
            </div>
          </div>
        </>
      )}

      {/* ===== 空港タブ ===== */}
      {activeTab === "airport" && (
        <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
          {airports.map(ap => {
            const apInfo = AIRPORT_URLS[ap.code];
            return (
              <div key={ap.code} style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", marginBottom: 8 }}>
                  <div>
                    <div style={{ color: "#e2e2ff", fontWeight: 700, fontSize: 16 }}>{ap.name}</div>
                    <div style={{ color: "#6b6b9b", fontSize: 12, marginTop: 2 }}>アクセス: {ap.lines}</div>
                  </div>
                  <span style={{ background: "#1e3a5f", color: "#60a5fa", fontSize: 12, fontWeight: 700, padding: "2px 8px", borderRadius: 4 }}>{ap.code}</span>
                </div>
                <div style={{ color: "#c084fc", fontSize: 12, marginBottom: 12, lineHeight: 1.6 }}>💡 {ap.tip}</div>
                {apInfo && (
                  <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 8 }}>
                    <button onClick={() => openUrl(apInfo.flightUrl)} style={{ background: "#0d0d1a", border: "1px solid #2d2d4e", color: "#e2e2ff", borderRadius: 8, padding: "10px 8px", cursor: "pointer", fontSize: 12, fontWeight: 700 }}>
                      <div style={{ fontSize: 18, marginBottom: 2 }}>✈️</div>
                      フライト情報
                    </button>
                    <button onClick={() => openUrl(apInfo.yahooUrl)} style={{ background: "#7c3aed", border: "none", color: "#fff", borderRadius: 8, padding: "10px 8px", cursor: "pointer", fontSize: 12, fontWeight: 700 }}>
                      <div style={{ fontSize: 18, marginBottom: 2 }}>🚉</div>
                      アクセス情報
                    </button>
                  </div>
                )}
              </div>
            );
          })}
        </div>
      )}

    </div>
  );
}

// ===== TIME TAB =====
function TimeTab() {
  const [shift, setShift] = useState("alternating_day");
  const [customStart, setCustomStart] = useState("07:00");
  const [customEnd, setCustomEnd] = useState("19:00");
  const [startedAt, setStartedAt] = useState(null);
  const [now, setNow] = useState(new Date());

  useEffect(() => { const t = setInterval(() => setNow(new Date()), 1000); return () => clearInterval(t); }, []);

  const shiftInfo = SHIFT_TYPES.find(s => s.id === shift);

  const parseTime = (str) => {
    const [h, m] = str.split(":").map(Number);
    return h * 60 + m;
  };

  const getElapsed = () => {
    if (!startedAt) return null;
    const diff = Math.floor((now - startedAt) / 1000);
    const h = Math.floor(diff / 3600);
    const m = Math.floor((diff % 3600) / 60);
    const s = diff % 60;
    return { h, m, s, total: diff };
  };

  const getShiftTotal = () => {
    if (shift === "custom") {
      const s = parseTime(customStart), e = parseTime(customEnd);
      return (e >= s ? e - s : 1440 - s + e) * 60;
    }
    return (shiftInfo?.hours || 12) * 3600;
  };

  const elapsed = getElapsed();
  const shiftTotal = getShiftTotal();
  const pct = elapsed ? Math.min(Math.round(elapsed.total / shiftTotal * 100), 100) : 0;

  const restGoal = (shiftTotal / 3600) >= 16 ? 3 : 1; // 16h以上は休憩3回推奨

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
      {/* シフト選択 */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>勤務体系を選択</div>
        <div style={{ display: "flex", flexDirection: "column", gap: 6 }}>
          {SHIFT_TYPES.map(s => (
            <button key={s.id} onClick={() => setShift(s.id)} style={{
              background: shift === s.id ? "#1e1e3f" : "#0d0d1a",
              border: shift === s.id ? "1px solid #7c3aed" : "1px solid #2d2d4e",
              color: shift === s.id ? "#c084fc" : "#8b8bbd",
              borderRadius: 8, padding: "10px 14px", cursor: "pointer", textAlign: "left", fontSize: 13, fontWeight: shift === s.id ? 700 : 400
            }}>
              {s.label} {s.start && s.end ? `（${s.start}〜${s.end}）` : ""}
            </button>
          ))}
        </div>
        {shift === "custom" && (
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 8, marginTop: 10 }}>
            <div>
              <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>開始時刻</div>
              <input type="time" value={customStart} onChange={e => setCustomStart(e.target.value)} style={inputStyle} />
            </div>
            <div>
              <div style={{ color: "#6b6b9b", fontSize: 11, marginBottom: 4 }}>終了時刻</div>
              <input type="time" value={customEnd} onChange={e => setCustomEnd(e.target.value)} style={inputStyle} />
            </div>
          </div>
        )}
      </div>

      {/* タイマー */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 16, border: "1px solid #2d2d4e", textAlign: "center" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 12, letterSpacing: 1 }}>勤務タイマー</div>
        {elapsed ? (
          <>
            <div style={{ color: pct >= 90 ? "#ef4444" : pct >= 70 ? "#f97316" : "#c084fc", fontSize: 44, fontWeight: 900, letterSpacing: 3, fontVariantNumeric: "tabular-nums" }}>
              {String(elapsed.h).padStart(2, "0")}:{String(elapsed.m).padStart(2, "0")}:{String(elapsed.s).padStart(2, "0")}
            </div>
            <div style={{ color: "#6b6b9b", fontSize: 13, marginTop: 4 }}>
              {shiftInfo?.hours || ""}h勤務のうち <span style={{ color: "#c084fc", fontWeight: 700 }}>{pct}%</span> 経過
            </div>
            <div style={{ background: "#0d0d1a", borderRadius: 6, height: 10, marginTop: 10, overflow: "hidden" }}>
              <div style={{ width: `${pct}%`, background: pct >= 90 ? "#ef4444" : "linear-gradient(90deg,#7c3aed,#c084fc)", height: "100%", borderRadius: 6, transition: "width 1s" }} />
            </div>
            <div style={{ color: "#8b8bbd", fontSize: 12, marginTop: 8 }}>
              推奨休憩: {restGoal}回 | 2時間に1回は必ず休憩を
            </div>
            <button onClick={() => setStartedAt(null)} style={{ marginTop: 14, background: "#3d1a1a", color: "#ef4444", border: "1px solid #ef4444", borderRadius: 8, padding: "10px 24px", fontWeight: 700, cursor: "pointer", fontSize: 14 }}>
              終了
            </button>
          </>
        ) : (
          <>
            <div style={{ color: "#4b4b7b", fontSize: 14, marginBottom: 16 }}>タイマーを開始してください</div>
            <button onClick={() => setStartedAt(new Date())} style={{ background: "linear-gradient(135deg,#7c3aed,#4f46e5)", color: "#fff", border: "none", borderRadius: 10, padding: "14px 32px", fontWeight: 700, fontSize: 16, cursor: "pointer", letterSpacing: 1 }}>
              🚖 勤務開始
            </button>
          </>
        )}
      </div>

      {/* 時間帯戦略 */}
      <div style={{ background: "#1a1a2e", borderRadius: 12, padding: 14, border: "1px solid #2d2d4e" }}>
        <div style={{ color: "#8b8bbd", fontSize: 12, marginBottom: 10, letterSpacing: 1 }}>時間帯別戦略</div>
        {[
          { time: "07-09時", type: "ラッシュ", tip: "通勤・駅への短距離多め。回転重視。" },
          { time: "10-16時", type: "閑散", tip: "観光エリア・病院・ショッピング施設狙い。" },
          { time: "17-20時", type: "帰宅ラッシュ", tip: "ビジネス街から居住区への中距離が出やすい。" },
          { time: "20-23時", type: "飲み需要", tip: "繁華街に張り付く。宴会帰りの郊外行き狙い。" },
          { time: "23-03時", type: "深夜ゴールデン", tip: "最高単価帯。新宿・六本木・渋谷に集中。" },
          { time: "03-07時", type: "夜明け", tip: "需要急減。空港行き・早出ビジネス客に絞る。" },
        ].map(item => (
          <div key={item.time} style={{ display: "flex", gap: 10, padding: "8px 0", borderBottom: "1px solid #1a1a3e" }}>
            <div style={{ minWidth: 60, color: "#c084fc", fontWeight: 700, fontSize: 12 }}>{item.time}</div>
            <div>
              <div style={{ color: "#e2e2ff", fontSize: 13, fontWeight: 600 }}>{item.type}</div>
              <div style={{ color: "#8b8bbd", fontSize: 12, lineHeight: 1.5 }}>{item.tip}</div>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

// ===== MAIN APP =====
const inputStyle = {
  background: "#0d0d1a", border: "1px solid #2d2d4e", color: "#e2e2ff",
  borderRadius: 8, padding: "9px 12px", fontSize: 13, width: "100%", boxSizing: "border-box",
};

const TABS = [
  { id: "sales", label: "売上", icon: "💴" },
  { id: "area", label: "エリア", icon: "🗺" },
  { id: "event", label: "イベント", icon: "🎆" },
  { id: "transport", label: "交通", icon: "🚃" },
  { id: "time", label: "時間", icon: "⏱" },
  { id: "settings", label: "設定", icon: "⚙️" },
];

function TaxiApp() {
  const [tab, setTab] = useState("sales");
  const [records, setRecords] = useState([]);
  const [zone, setZone] = useState("tokubestu");
  const [closeDay, setCloseDay] = useState(15);
  const [defaultShift, setDefaultShift] = useState("alternating_day");

  // ストレージから初期データ読み込み
  useEffect(() => {
    (async () => {
      const savedRecords = await storage.get("taxi_records");
      if (savedRecords) { try { setRecords(JSON.parse(savedRecords)); } catch {} }
      const savedZone = await storage.get("taxi_zone");
      if (savedZone && ALL_ZONES[savedZone]) setZone(savedZone);
      const savedCloseDay = await storage.get("taxi_close_day");
      if (savedCloseDay) { const n = Number(savedCloseDay); if (n >= 1 && n <= 28) setCloseDay(n); }
      const savedShift = await storage.get("taxi_default_shift");
      if (savedShift) setDefaultShift(savedShift);
    })();
  }, []);

  const zoneInfo = ALL_ZONES[zone];

  return (
    <div style={{ background: "#0a0a1a", minHeight: "100vh", maxWidth: 480, margin: "0 auto", fontFamily: "'Noto Sans JP', 'Hiragino Sans', sans-serif", paddingBottom: 80 }}>
      {/* Header */}
      <div style={{ background: "linear-gradient(135deg, #1a0a2e 0%, #0a0a1a 100%)", padding: "16px 16px 10px", borderBottom: "1px solid #2d2d4e", position: "sticky", top: 0, zIndex: 100 }}>
        <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
          <div>
            <div style={{ color: "#c084fc", fontWeight: 900, fontSize: 18, letterSpacing: 1 }}>🚖 TAXI MANAGER</div>
            <div style={{ color: "#4b4b7b", fontSize: 11, marginTop: 2 }}>底辺タクシー FIRE挑戦中</div>
          </div>
          <div
            onClick={() => setTab("settings")}
            style={{ background: "#1a1a3e", border: "1px solid #3d3d6e", borderRadius: 8, padding: "5px 10px", cursor: "pointer", textAlign: "right" }}
          >
            <div style={{ color: "#c084fc", fontSize: 10, fontWeight: 700, letterSpacing: 0.5 }}>営業圏</div>
            <div style={{ color: "#e2e2ff", fontSize: 11, fontWeight: 700, marginTop: 1 }}>{zoneInfo?.short || "未設定"}</div>
          </div>
        </div>
      </div>

      {/* Content */}
      <div style={{ padding: "16px" }}>
        {tab === "sales" && <SalesTab records={records} setRecords={setRecords} closeDay={closeDay} defaultShift={defaultShift} />}
        {tab === "area" && <AreaTab zone={zone} />}
        {tab === "event" && <EventTab zone={zone} />}
        {tab === "transport" && <TransportTab zone={zone} />}
        {tab === "time" && <TimeTab />}
        {tab === "settings" && <SettingsTab zone={zone} setZone={setZone} closeDay={closeDay} setCloseDay={setCloseDay} defaultShift={defaultShift} setDefaultShift={setDefaultShift} />}
      </div>

      {/* Bottom Nav */}
      <div style={{ position: "fixed", bottom: 0, left: "50%", transform: "translateX(-50%)", width: "100%", maxWidth: 480, background: "#0d0d1a", borderTop: "1px solid #2d2d4e", display: "flex", zIndex: 200 }}>
        {TABS.map(t => (
          <button key={t.id} onClick={() => setTab(t.id)} style={{
            flex: 1, background: "none", border: "none", cursor: "pointer", padding: "10px 0",
            display: "flex", flexDirection: "column", alignItems: "center", gap: 2,
          }}>
            <span style={{ fontSize: 20 }}>{t.icon}</span>
            <span style={{ fontSize: 10, color: tab === t.id ? "#c084fc" : "#4b4b7b", fontWeight: tab === t.id ? 700 : 400 }}>{t.label}</span>
            {tab === t.id && <div style={{ width: 20, height: 2, background: "#c084fc", borderRadius: 2 }} />}
          </button>
        ))}
      </div>
    </div>
  );
}


    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<TaxiApp />);
  </script>
</body>
</html>
