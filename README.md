# moNa2 v2 (DYA Studio 対応版)

このリポジトリは [moNa2 v2](https://github.com/sayu-hub/zmk-config-moNa2) 用 ZMK Config を、[cormoran](https://github.com/cormoran) さんが公開している **[DYA Studio](https://studio.dya.cormoran.works/)** に対応させたものです。

DYA Studio は ZMK Studio をベースに、

- キーマップ／レイヤーの GUI 編集
- **マクロの作成・編集・削除（ファームウェア再ビルド不要）**
- **コンボの追加・編集・削除（ファームウェア再ビルド不要）**
- トラックボール (PMW3610) のスクロール量・軸反転・自動マウスレイヤーなど **Runtime Input Processor 経由での動的設定**
- BLE プロファイル管理 / Settings RPC

を Web から行えるようにした拡張版です。

## セットアップ方法

### 組み立て済みのmoNa2 v2を購入した場合

DYA Studio対応ファームウェアは書き込み済みです。ファームウェアのビルドや書き込みは必要ありません。そのまま「[DYA Studio の使い方](#dya-studio-の使い方)」へ進み、キーマップを編集してください。

### ファームウェアを書き込む場合

通常はソースコードをFork／ビルドする必要はありません。

1. **[DYA Studio対応ファームウェア v1.0.0](https://github.com/shakushakupanda/zmk-config-moNa2-v2/releases/tag/dya-studio-v1.0.0)** を開きます。
2. **Assets** から必要なUF2ファイルをダウンロードします。ZIP形式の場合は展開します。
3. 右手側は、搭載しているトラックボールセンサーに合うUF2ファイルを選びます。

| ファイル | 用途 |
| --- | --- |
| `mona2_r-pmw3610.uf2` | 右手側・PMW3610版 |
| `mona2_r-paw3222.uf2` | 右手側・PAW3222版 |
| `mona2_l-*.uf2` | 左手側 |
| `settings_reset-*.uf2` | 保存済み設定の初期化 |

左右それぞれで以下を行います。

1. USBケーブルでPCへ接続します。
2. XIAO付近のカバーを上から素早く2回押し、内部のリセットスイッチを操作してブートローダーモードにします（[操作動画](https://x.com/shakupan_/status/1966705930038600134?s=20)）。
3. PCに表示されたUSBドライブへ、対応するUF2ファイルをドラッグ＆ドロップします。
4. 自動的に再起動したら書き込み完了です。

左右で異なるバージョンのファームウェアを使用すると、分割通信が正常に動作しないことがあります。左右とも同じビルドのファームウェアを書き込んでください。

### 既存ファームウェアから更新するとき

ZMK v0.4系への移行により、以前のキーマップ、トラックボール設定、BLE情報は引き継がれません。次の手順で初期化してください。

1. OS側で既存のmoNa2のBluetooth登録を削除します。
2. 左右それぞれに `settings_reset-*.uf2` を書き込みます。
3. 左右それぞれに通常のファームウェアを書き直します。
4. Bluetoothを再ペアリングします。

設定リセットを行うと、保存済みキーマップやBluetoothペアリング情報は消去されます。

### ファームウェア自体を変更する場合

キーマップやDYA Studioから変更できる設定だけを編集する場合、Forkは不要です。ソースコード、デフォルトキーマップ、機能、ビルド設定など、ファームウェア自体を変更したい場合のみ次の手順でビルドします。

1. このリポジトリを **Fork** します。
2. Fork先の **Actions** タブを開きます。
3. GitHub Actionsが無効の場合は、**I understand my workflows, go ahead and enable them** を押して有効化します。
4. 必要なファイルを編集してcommitします。
5. `.github/workflows/build.yml` のワークフローを選択します。
6. **Run workflow** を押して `main` ブランチのビルドを実行します。
7. ビルド完了後、実行結果の **Artifacts** から `firmware` をダウンロードして書き込みます。

## DYA Studio の使い方

1. 中央側（右手）を **USBケーブル** でPCに接続します。
2. Chrome / EdgeなどのWebUSB対応ブラウザで **[DYA Studio](https://studio.dya.cormoran.works/)** を開きます。
3. キーボード側で `BLE` レイヤー（レイヤー10/11）を有効にし、右下に配置した **`&studio_unlock`** キーを押してアンロックします。
4. DYA Studio側で **Connect** を押し、USBデバイスとしてmoNa2を選択します。
5. キーマップ／マクロ／コンボ／トラックボール設定を編集します。

### マクロを DYA Studio から使う

マクロのランタイム編集は [zmk-feature-runtime-macro](https://github.com/cormoran/zmk-feature-runtime-macro) が担当します。

1. DYA Studio のマクロ画面で新しいマクロを作成し、名前と内容を設定して保存します。
2. マクロ一覧に表示される **スロット番号 (0〜7)** を確認します。
3. DYA Studio のキーマップエディタで、任意のキーに `&rmacro <スロット番号>` を割り当てます。

キーマップ側にはすでに `#include <behaviors/runtime_macro.dtsi>` を入れてあるので、`&rmacro` は最初から選択できます。

既存の `&BT0` / `&screenshot` / `&henkan` などは従来どおりの **静的マクロ** のままで、DYA Studio からは編集できません（キーマップに直接名前で書かれているため、そのまま残しています）。編集したくなったら DYA Studio 側で同じ内容のランタイムマクロを作り直してキーを差し替えてください。

### コンボを DYA Studio から使う

コンボは [zmk-feature-runtime-combo](https://github.com/cormoran/zmk-feature-runtime-combo) に移行済みです。

`config/mona2.keymap` の `runtime_combo_defaults` ノードが **コンパイル時デフォルト**（フラッシュ直後から有効・設定消去でも復活）で、現状は次の 2 つが入っています。

| スロット | 内容 | キー位置 |
| --- | --- | --- |
| 0 | `&lt 4 ESC` | 38, 39 |
| 1 | `&kp TAB` | 11, 12 |

スロットは全部で 16 個 (`CONFIG_ZMK_RUNTIME_COMBO_MAX_COMBOS`) あるので、残り 14 個は DYA Studio から自由に追加できます。既存スロットを Studio 上で書き換えた場合は、Web UI の **Reset to Default** でここの値へ戻せます。

### このリポジトリで対応済みの内容

| 対応項目 | ファイル / 箇所 |
| --- | --- |
| ZMK 本体を cormoran さんの fork (`main+dya`, ZMK v0.4 / Zephyr 4.1 系) に切替 | `config/west.yml` |
| DYA Studio 基盤モジュール (custom-settings / ble-management / settings-rpc / runtime-input-processor / runtime-sensor-rotate) | `config/west.yml` |
| **マクロ・コンボのランタイム編集モジュール (runtime-macro / runtime-combo) を追加** | `config/west.yml` |
| Studio 系 CONFIG (`CONFIG_ZMK_STUDIO`, `CONFIG_ZMK_CUSTOM_SETTINGS`, `CONFIG_ZMK_BLE_MANAGEMENT`, `CONFIG_ZMK_SETTINGS_RPC`, `CONFIG_ZMK_RUNTIME_INPUT_PROCESSOR`, `CONFIG_ZMK_RUNTIME_MACRO`, `CONFIG_ZMK_RUNTIME_COMBO` ほか) | `config/mona2_r.conf` |
| 静的コンボ (`zmk,combos`) を `runtime_combo_defaults` へ移行 | `config/mona2.keymap` |
| `&rmacro` (ランタイムマクロ behavior) を利用可能に | `config/mona2.keymap` |
| **DYA Studio 診断タブ用モジュール (kscan-diagnostics / input-stream / devtool) を追加** | `config/west.yml`, `config/mona2_r.conf`, `config/mona2_l.conf` |
| **トラックボールドライバを cormoran さんの Studio RPC 対応版へ移行** | `config/west.yml`, `mona2_r.overlay`, `Kconfig.defconfig`, `mona2_r.conf` |
| トラックボール処理を Runtime Input Processor に置換 | `boards/shields/mona2/mona2.dtsi`, `mona2_r.overlay` |
| `&studio_unlock` を `ble_win` / `ble_mac` レイヤー右上に配置 | `config/mona2.keymap` |
| 全レイヤーに `display-name` を設定 | `config/mona2.keymap` |
| `studio-rpc-usb-uart` snippet (中央側) / ボード名を `xiao_ble/nrf52840/zmk` へ | `build.yaml` |

> **Note:**
> - DYA Studio で行った変更は **中央側 (右手) の Flash** に保存されます。初期化したい場合は `settings_reset` ファームウェアを書き込んでください。
> - **ZMK v0.4 系への移行に伴い、以前の設定（保存済みキーマップ・トラックボール設定）は引き継がれません。** 書き込み後に DYA Studio から設定し直してください。

## ZMK v0.4 系への移行で変わった点

`v0.3-branch+dya` (Zephyr 3.5) から `main+dya` (Zephyr 4.1) に上げたことで、以下の破壊的変更に追従しています。

| 変更 | 対応 |
| --- | --- |
| ボード名 `seeeduino_xiao_ble` → `xiao_ble`、ZMK 用バリアント `/nrf52840/zmk` が必須 | `build.yaml` |
| Zephyr 4.1 が純正 `pixart,pmw3610` ドライバを同梱したため、旧 compatible `pixart,pmw3610` が使えなくなった | トラックボールドライバを cormoran さんの版へ移行（下記参照） |
| `zmk-rgbled-widget` の `v0.3` ブランチは Zephyr 3.5 向け | `west.yml` (`v0.3` → `main`) |
| endpoints API が `zmk_endpoints_send_mouse_report()` → `zmk_endpoint_send_mouse_report()` に改名 | `zmk-module-mouse-gesture-rpc` の `main` に取り込み済み |
| 再利用 GitHub Actions ワークフローが `@v0.3.0` のままだと Zephyr 4.1 でビルドできない | `.github/workflows/build.yml` (`@main`) |

## DYA Studio の診断タブ対応

DYA Studio の診断ページには専用モジュールが必要なパネルがあり、
未対応だと「このキーボードでは利用できません」と表示されます。以下を有効にしています。

| DYA Studio のパネル | モジュール | 主な CONFIG |
| --- | --- | --- |
| キースイッチ（キー押下統計・チャタリング検出） | [zmk-feature-kscan-diagnostics](https://github.com/cormoran/zmk-feature-kscan-diagnostics) | `CONFIG_ZMK_KSCAN_DIAGNOSTICS` |
| ↑のライブ表示 | [zmk-feature-input-stream](https://github.com/cormoran/zmk-feature-input-stream) | `CONFIG_ZMK_INPUT_STREAM_FEATURE` |
| トラックボールセンサー（PMW3610 の状態・表面診断） | [zmk-driver-pmw3610-with-custom-studio-rpc](https://github.com/cormoran/zmk-driver-pmw3610-with-custom-studio-rpc) | `CONFIG_ZMK_PMW3610_STUDIO_RPC` |
| スタック使用量 | [zmk-module-devtool](https://github.com/cormoran/zmk-module-devtool) | `CONFIG_ZMK_DEVTOOL_STACK_USAGE` |
| デバイス情報（ビルド・ハードウェア・ランタイム詳細） | [zmk-feature-device-info](https://github.com/cormoran/zmk-feature-device-info) | `CONFIG_ZMK_DEVICE_INFO` |
| 安定性（ウォッチドッグ） | [zmk-feature-watchdog](https://github.com/cormoran/zmk-feature-watchdog) | `CONFIG_ZMK_WATCHDOG` |
| Zephyr settings の閲覧 | [zmk-feature-zephyr-setting-expose](https://github.com/cormoran/zmk-feature-zephyr-setting-expose) | `CONFIG_ZMK_SETTING_EXPOSE` |
| Studio RPC の性能計測 | [zmk-feature-studio-rpc-perf](https://github.com/cormoran/zmk-feature-studio-rpc-perf) | `CONFIG_ZMK_STUDIO_RPC_PERF` |
| OS ごとのデフォルトレイヤー | [zmk-feature-default-layer](https://github.com/cormoran/zmk-feature-default-layer) + [zmk-feature-os-detection](https://github.com/cormoran/zmk-feature-os-detection) | `CONFIG_ZMK_DEFAULT_LAYER`, `CONFIG_ZMK_OS_DETECTION` |

ウォッチドッグはフリーズ・ハードフォルト・予期しないリセットを Flash に記録します。
左右それぞれが自分のインシデントを記録し、Web UI の source セレクタで
「Central」「Peripheral」を切り替えて参照できます。
なお `CONFIG_ZMK_WATCHDOG_FREEZE_DETECT`（既定で有効）は、システムワークキューが
10 秒以上詰まった場合にインシデントを記録して再起動します。
この挙動が不要なら `CONFIG_ZMK_WATCHDOG_FREEZE_DETECT=n` を追加してください。

### OS ごとのデフォルトレイヤー

接続先の OS（USB / BLE プロファイルごと）を判定して、起動時のデフォルトレイヤーを
自動で切り替えられます。mona2 はベースレイヤーが 0 = `WIN` / 1 = `MAC` に分かれているので、
選択可能な範囲を `CONFIG_ZMK_DEFAULT_LAYER_MIN_INDEX=0` /
`CONFIG_ZMK_DEFAULT_LAYER_MAX_INDEX=1` に設定しています。
実際の割り当ては DYA Studio の「OSごとのデフォルトレイヤー」パネルから行ってください。

キーマップには `&df` behavior も入れてあるので、DYA Studio のキーマップエディタから
`&df DF_SEL <レイヤー>`（指定レイヤーをデフォルトに）や `&df DF_INC`（次のレイヤーへ）を
任意のキーに割り当てることもできます。

> os-detection 自身の `CONFIG_ZMK_OS_DETECTION_LAYER_*` による自動切替は
> default-layer モジュールと競合するため有効にしないでください。

`zmk-feature-default-layer` だけは `main` ではなく `codex/custom-rpc-rewrite` ブランチを
使っています。`main` には DYA Studio 用の custom Studio RPC が入っておらず、
パネルから操作できないためです（DYA2 v2.0 も同じブランチを使用）。

キースイッチ診断は分割キーボードなので、周辺側（左手）の統計を中央側経由で取得するために
`CONFIG_ZMK_SPLIT_RELAY_EVENT` を有効にし、`CONFIG_ZMK_SPLIT_RELAY_EVENT_DATA_LEN=256` を
**左右両方の conf に同じ値で**設定しています。

### トラックボールドライバの移行について

「トラックボールセンサー」パネルは cormoran さんのドライバでしか動かないため、
badjeff さんの `zmk-pmw3610-driver` から
`zmk-driver-pmw3610-with-custom-studio-rpc` へ移行しました。

- devicetree の compatible が `pixart,pmw3610-alt` → **`cormoran,pmw3610`**
- **軸の反転は devicetree プロパティではなく Kconfig になりました。**
  `invert-x;` は `CONFIG_PMW3610_INVERT_X=y` に移動しています
  （COROPIT 版などで Y も反転したい場合は `CONFIG_PMW3610_INVERT_Y=y`）。
- CPI・軸反転・ダウンシフト時間などは `CONFIG_ZMK_PMW3610_CUSTOM_SETTINGS=y` により
  **DYA Studio から実行時に変更・保存できます**。
- ドライバ実装が変わるため、カーソルの追従感やスリープ復帰の挙動が
  以前と微妙に変わる可能性があります。違和感があれば DYA Studio 側で
  CPI やダウンシフト時間を調整してみてください。

PAW3222 版ビルドは従来どおり sekigon-gonnoc さんのドライバを使うため影響ありません
（`config/paw3222.overlay` で compatible を上書きしています）。

### 慣性スクロール

マウスジェスチャーの慣性スクロール（スクロールを止めたあと惰性で流れる挙動）は
**既定で無効**です。使いたい場合は DYA Studio の設定画面から有効化してください
（有効にした設定はフラッシュに保存され、次回起動時にも復元されます）。

有効/無効の既定値は `zmk-module-mouse-gesture-rpc` 側でしか変えられません
（入力プロセッサの初期値と設定ストアの初期値の両方にハードコードされているため）。
現在はモジュールの `main` で既定オフになっています。

> 完全に無効でよいなら、`mona2_r.overlay` の `&trackball_central_listener` の
> `input-processors` から `&inertial_scroll` を外す方法もあります。
> ただしその場合は DYA Studio からも有効化できなくなります。

### mouse-gesture-rpc の参照先について

`zmk-module-mouse-gesture-rpc` は、ファームウェア側の開発ブランチ
`expose-mg-set-to-studio` を `main` にマージ済みのため、`main` を参照しています。

ZMK v0.4 で消えた `zmk_endpoints_send_mouse_report()` の改名対応と、
慣性スクロールを既定オフにする変更はモジュール側に取り込まれているので、
以前このリポジトリに置いていた互換シム（`compat/`、`CMakeLists.txt`、
`zephyr/module.yml` の `cmake: .`）は不要になり削除しました。

Web UI の数値入力欄が最初の1桁で固まる問題の修正差分
（`docs/mouse-gesture-web-numberfield.patch`）も `main` へ取り込み済みです。

### PAW3222 版ビルドについて（既存不具合の修正）

Zephyr は devicetree を Kconfig より **先に** 処理するため、`mona2_r.overlay` 内の
`#ifdef CONFIG_TRACKBALL_PAW3222` はクリーンビルドでは常に偽になります。
その結果、これまで `mona2_r-paw3222` という名前で生成されていた成果物は
**実際には PMW3610 版**になっていました。

`config/paw3222.overlay` を追加し、`build.yaml` から `-DEXTRA_DTC_OVERLAY_FILE` で
渡すことで、正しく `pixart,paw3222` が選ばれるよう修正しています。

### ビルド検証状況

このブランチは実機書き込み前に、ローカル (Zephyr SDK 0.17.0 / arm-zephyr-eabi) で
以下 4 構成のビルドが通ることを確認しています。

| 構成 | 結果 | FLASH | RAM |
| --- | --- | --- | --- |
| `mona2_r rgbled_adapter` (PMW3610) | OK | 51.23% | 65.77% |
| `mona2_r rgbled_adapter` (PAW3222) | OK | 50.53% | 65.43% |
| `mona2_l rgbled_adapter` | OK | 27.14% | 25.06% |
| `settings_reset` | OK | 7.31% | 6.66% |

トラックボール、マウスジェスチャー、エンコーダ、BLE ペアリングを含む実機動作を確認済みです。

---
