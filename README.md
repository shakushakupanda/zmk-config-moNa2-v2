# moNa2 v2 firmware

moNa2 v2 用の ZMK ファームウェア設定です。

> [!IMPORTANT]
> **最新・推奨版は [`dya-studio` ブランチ](https://github.com/shakushakupanda/zmk-config-moNa2-v2/tree/dya-studio) です。**
>
> DYA Studioによるキーマップ、マクロ、コンボ、トラックボール設定などのリアルタイム編集に対応しています。新しく導入する場合は `dya-studio` ブランチを使用してください。

この `main` ブランチは、ZMK v0.3系を使用した従来版として残しています。

## セットアップ方法

### 1. 使用するブランチを選ぶ

- 新規導入・DYA Studioを利用する場合：[`dya-studio`](https://github.com/shakushakupanda/zmk-config-moNa2-v2/tree/dya-studio)
- 従来のZMK v0.3系を継続利用する場合：`main`

### 2. リポジトリをForkする

1. このページ右上の **Fork** を押します。
2. `dya-studio` を使用する場合は、Fork作成画面の **Copy the `main` branch only** のチェックを外し、全ブランチをForkします。
3. Fork先のリポジトリで **Actions** タブを開きます。
4. GitHub Actionsが無効の場合は、**I understand my workflows, go ahead and enable them** を押して有効化します。

### 3. ファームウェアをビルドする

1. Fork先の **Actions** タブを開きます。
2. `.github/workflows/build.yml` のワークフローを選択します。
3. **Run workflow** を押します。
4. 使用するブランチに `dya-studio` または `main` を指定して実行します。
5. ビルド完了後、実行結果の **Artifacts** から `firmware` をダウンロードして展開します。

`dya-studio` ブランチでは、主に次のUF2ファイルが生成されます。

| ファイル | 用途 |
| --- | --- |
| `mona2_r-pmw3610.uf2` | 右手側・PMW3610版 |
| `mona2_r-paw3222.uf2` | 右手側・PAW3222版 |
| `mona2_l-*.uf2` | 左手側 |
| `settings_reset-*.uf2` | 保存済み設定の初期化 |

右手側は、搭載しているトラックボールセンサーに合うファイルを選んでください。

### 4. ファームウェアを書き込む

左右それぞれで以下を行います。

1. USBケーブルでPCへ接続します。
2. リセットボタンを素早く2回押し、ブートローダーモードにします。
3. PCに表示されたUSBドライブへ、対応するUF2ファイルをドラッグ＆ドロップします。
4. 自動的に再起動したら書き込み完了です。

左右で異なるバージョンのファームウェアを使用すると、分割通信が正常に動作しないことがあります。左右とも同じブランチ・同じビルドのファームウェアを書き込んでください。

### 5. DYA Studioを使用する

`dya-studio` ブランチを書き込んだ場合は、右手側をUSB接続して [DYA Studio](https://studio.dya.cormoran.works/) を開きます。詳しい接続方法と機能については、[`dya-studio` ブランチのREADME](https://github.com/shakushakupanda/zmk-config-moNa2-v2/tree/dya-studio#dya-studio-%E3%81%AE%E4%BD%BF%E3%81%84%E6%96%B9) を参照してください。

## 既存ファームウェアから更新するとき

ZMKのバージョンや保存形式が変わる場合、以前のキーマップ、トラックボール設定、BLE情報が正しく引き継がれないことがあります。動作が不安定な場合は、次の手順で初期化してください。

1. OS側で既存のmoNa2のBluetooth登録を削除します。
2. 左右それぞれに `settings_reset-*.uf2` を書き込みます。
3. 左右それぞれに通常のファームウェアを書き直します。
4. Bluetoothを再ペアリングします。

設定リセットを行うと、保存済みキーマップやBluetoothペアリング情報は消去されます。

## COROPITを使用する場合（mainブランチ）

`main` ブランチでCOROPITを使用する場合は、`boards/shields/mona2/mona2_r.overlay` の `trackball_central` を次のように変更してください。

```dts
trackball_central: trackball_central@0 {
    status = "okay";
    compatible = "pixart,pmw3610";
    reg = <0>;
    spi-max-frequency = <2000000>;
    irq-gpios = <&gpio0 2 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
    cpi = <600>;
    //swap-xy;
    invert-x;
    invert-y;
    evt-type = <INPUT_EV_REL>;
    x-input-code = <INPUT_REL_X>;
    y-input-code = <INPUT_REL_Y>;
};
```

`dya-studio` ブランチでは軸反転をDYA StudioまたはKconfigから設定できるため、この編集方法は対象外です。
