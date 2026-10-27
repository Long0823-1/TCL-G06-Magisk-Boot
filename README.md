# TCL G06 / MT5867 Magisk適用済みbootイメージ

TCL 32S5400（G06 / MT5867）向け、Magisk **25.2（25200）** 適用済みのbootパーティションバックアップです。公式ファームウェアではありません。

## 実機検証：公開bootをFastbootから書き込んで正常起動（2026-10-27）

ユーザーの実機で、次の流れによる**公開Magisk適用済みbootの書き込みと通常起動**を確認しました。MBOOTからのboot書き込みを使わずに起動できた検証結果です。

1. 対象TVに対応するStock USBで純正状態へ復元。
2. ADBを有効化し、Fastbootへ移行。
3. `fastboot flashing unlock` でブートローダーをアンロック。
4. vbmetaの検証を無効化。
5. 公開ファイル `boot-magisk-25.2-g06.img` をFastbootからbootへ書き込み。
6. `fastboot erase userdata` を実行。
7. `fastboot reboot` で再起動し、正常な通常起動を確認。

実際に報告されたboot書き込み以降のコマンドは次のとおりです。

```powershell
fastboot flash boot "C:\Users\Owner\Downloads\boot-magisk-25.2-g06.img"
fastboot erase userdata
fastboot reboot
```

**userdata消去を含む試験です。設定・アプリ・アプリデータは失われます。** アンロック時の消去にも備え、必要なデータを退避してください。vbmeta無効化時に実行したコマンド全文は報告に含まれていないため、下記のオプション付きコマンドは操作例として掲載しています。

この記録で新たに確認できた範囲は、上記条件でのFastboot書き込みと通常起動です。`su -c id`の出力や他個体・別ファームウェア、userdataを消去しない場合まで検証済みとはしていません。起動後に `adb shell "su -c id"` を実行し、Magiskの権限許可後に `uid=0(root)` が返ることを確認できます。

## ダウンロード

[Releases](../../releases) の `magisk-25.2-20261027` から `boot-magisk-25.2-g06.img` と `SHA256SUMS.txt` を取得してください。

| 項目 | 内容 |
|---|---|
| 対象 | TCL 32S5400 / G06 / MT5867、32bit ARM |
| Magisk | 25.2 / 25200 |
| イメージ | Android boot image header v2 / page size 2048 |
| サイズ | 32,505,856 bytes |
| SHA-256 | `ce9acfddb98cc1e27e09bde687114df13559eb0adc7e419b35c77276f2ec4747` |
| 実機動作 | **ユーザーによる起動試験済み・正常動作（2026-10-27報告）** |
| 他個体・別ファームウェア | 未検証。汎用イメージではありません |

元のbootパーティションバックアップをバイト変更せず、配布用の名前で収録しています。純正未改変bootではありません。ファームウェアの正確なビルド番号・fingerprintは今回の入力boot単体から確認できていないため、対応機種名だけで適合を判断しないでください。

## 確認した内容

- gzip ramdiskを展開し、CPIOを正常に読み取り。
- `.backup/.magisk`、元のinitバックアップ、Magisk32を確認。
- initは手元のMagisk 25.2配布物とSHA-256が一致。
- カーネル部分は保存済みの純正bootと一致。
- 今回の公開作業ではTVへの書き込み・再起動試験を実施していません。上記の実機正常動作はユーザーからの確認報告です。

パッチ設定は `KEEPVERITY=false`、`KEEPFORCEENCRYPT=false`、`PATCHVBMETAFLAG=false`、`RECOVERYMODE=false` です。これだけで現在のTVの暗号化状態やブートローダーのアンロック状態は判定できません。

## 導入前に

**対象TVと同じファームウェアの純正boot・vbmeta等を自分でバックアップし、復旧手段を確保してください。** ブートローダーのアンロックと、改変イメージに対応するAVB設定が前提です。アンロック時のデータ消去や、書き込み失敗による起動不能の可能性があります。改変状態で再ロックしないでください。

vbmetaの検証無効化手順は、[「vbmetaの検証無効化」](#vbmetaの検証無効化)を参照してください。vbmetaは同じファームウェアのものを使い、他機種の空vbmetaを流用しないでください。同じReleaseに元の`vbmeta.img`と`vbmeta_system.img`も添付しています。どちらも **Flags=0（検証無効化なし）** のバックアップで、無効化済みイメージではありません。`vbmeta_system`を一律に書き換える必要はありません。

## vbmetaの検証無効化

**改変bootを書き込む前に、必ずvbmetaのAVB検証を無効化してください。** 以下はWindowsのPowerShellで実行する手順です。TVへ実際に書き込む操作を含みます。

### 1. 何を無効化するのか

vbmetaにはboot等の検証に用いる署名・ハッシュ・検証設定が入っています。ブートローダーのアンロックと、vbmetaの検証無効化は別の操作です。Magisk適用済みbootを書き込んでも、別パーティションのvbmetaは自動で変更されません。

| 対象 | 配布時の状態 / 操作 |
|---|---|
| `vbmeta.img` | Flags=0。下記のFastbootコマンドで、書き込みデータの検証無効化フラグを設定します |
| `vbmeta_system.img` | Flags=0のバックアップ。以下の手順では書き換えません |
| ブートローダー | あらかじめアンロックが必要。vbmetaだけではアンロックできません |

`--disable-verity` はhashtree検証の無効化フラグ（1）、`--disable-verification` はAVB検証の無効化フラグ（2）を設定します。両方指定したトップレベルvbmetaのFlagsは3になります。PC上の元ファイルは変更されず、Fastbootが転送前に書き込みデータへ反映します。**元ファイルをもう一度調べてFlags=0でも、それだけで書き込み失敗とは判断できません。**

### 2. 対応ファイルを用意し、バックアップを保管する

- 自分のTVの純正boot・vbmeta等のバックアップと、対応するStock USBによる復旧手段を確保します。
- 同じファームウェアの `vbmeta.img` を使います。Releaseにも確認機のバックアップを添付していますが、機種名だけで別ファームウェアへの適合は判断できません。
- Releaseの `vbmeta.img` と `SHA256SUMS.txt` をダウンロードします。別機種のファイルや空のvbmetaは使いません。
- PCにはAndroid SDK Platform-Toolsの `adb` と `fastboot` を用意します。以下は両コマンドをPATHから実行できる例です。PowerShellで現在のフォルダーにあるexeを使う場合は `fastboot` を ` .\fastboot.exe` に読み替えます。

公式配布: [Android SDK Platform-Tools](https://developer.android.com/tools/releases/platform-tools)

例として `C:\Users\Owner\Downloads\vbmeta.img` に保存した場合、次を実行します。自分で採取したvbmetaは、自分のバックアップ時のハッシュと照合してください。

```powershell
Get-Item "C:\Users\Owner\Downloads\vbmeta.img" | Select-Object Name, Length
Get-FileHash "C:\Users\Owner\Downloads\vbmeta.img" -Algorithm SHA256
```

このReleaseのファイルの場合、サイズは **1,048,576 bytes**、SHA-256は次の値です。一致しなければ先に進まないでください。

```text
f18825b4d6f8ea2e51345f512ad25cb7d3d73ef73f5224c758b70876fbefa663
```

### 3. 対象TVをFastbootへ移し、アンロック状態を確認する

初回の `fastboot flashing unlock` は前提となる別操作です。データ消去が発生する場合があるため、必要なデータを退避し、TV側に確認画面が出た場合は表示内容を確認して操作してください。すでにアンロック済みなら、確認のためだけに再実行する必要はありません。

Androidが起動している状態でADBを有効化し、接続認証を許可してから実行します。

```powershell
adb devices
adb reboot bootloader
fastboot devices
```

ADBが複数接続されている場合は、最初の `adb devices` に表示された対象TVのシリアルを使い、`adb -s <ADB_SERIAL> reboot bootloader` と指定します。複数のFastboot端末がある場合も `fastboot -s <FASTBOOT_SERIAL> ...` と指定してください。ADBとFastbootのシリアルは同じとは限りません。出力が空・別端末なら書き込まないでください。

アンロック状態を確認する補助コマンドは次のとおりです。機種によっては未対応です。

```powershell
fastboot getvar unlocked
```

`unlocked: yes` を返す機種ではそれを確認します。未対応のエラーはアンロック済みの証明にはなりません。確認機では事前に `fastboot flashing unlock` を実行した流れで、公開bootの正常起動が報告されています。

### 4. 検証無効化オプションを付けてvbmetaを書き込む

対象TVだけを認識し、ファイルの適合・バックアップ・アンロックを確認してから、次を実行します。

```powershell
fastboot --disable-verity --disable-verification flash vbmeta "C:\Users\Owner\Downloads\vbmeta.img"
```

**両オプションを省略しないでください。** `fastboot flash vbmeta vbmeta.img` だけでは、今回添付したFlags=0のファイルを検証無効化せずに書くことになります。書き込み先はトップレベルの `vbmeta` です。本機はA-onlyであり、他機種の `vbmeta_a` / `vbmeta_b` 等へ勝手に置き換えないでください。

### 5. 成功を確認してからbootへ進む

通常は、転送（`Sending`）と書き込み（`Writing`）の両方で `OKAY` が表示されます。PowerShellでは、直後の終了コードも確認できます。

```powershell
$LASTEXITCODE
# 0であることを確認
```

`FAILED`、転送・書き込みエラー、0以外の終了コードが出たら中止します。`OKAY`は転送・書き込み処理の成功を示しますが、これだけでTV上のAVB挙動やroot取得まで証明するものではありません。実機のFlagsを厳密に確かめるには、書き込み後のvbmetaを読み戻して確認する必要があります。

成功後は、下の「Magisk適用済みbootの書き込み」へ進みます。今回の起動成功例は `fastboot erase userdata` も実行したものです。この操作ではTVの設定・アプリ・アプリデータが消えるので、冒頭の検証手順とデータ消去の注意を必ず確認してください。

### エラーが出た場合

| 症状 | 確認すること |
|---|---|
| `waiting for any device` / `fastboot devices` が空 | Fastbootモード、データ通信対応USBケーブル、PC側ドライバーを確認。ADB接続の成功だけではFastboot接続は確認できません |
| `Flashing is not allowed` / lockedに関する拒否 | ブートローダーのアンロックが完了しているか確認 |
| `unknown option` | 使用しているFastbootのパスと版を確認し、公式Platform-Toolsを使用 |
| `cannot load` / ファイルが見つからない | 保存先とファイル名を確認。パスに空白がある場合は引用符で囲む |
| partition不明・書き込み失敗 | 対象機・Fastbootモード・パーティション構成を確認。別の領域へ推測で書き込まない |

vbmetaの消去、空イメージの流用、`vbmeta_system`の一律書き換えで対処しないでください。MBOOTの `mmc write.p` にFastbootのオプションを付けても、この処理にはなりません。**検証無効化は起動時の改変検知を弱めます。改変状態のままブートローダーを再ロックしないでください。**

検証範囲：ユーザー実機では「vbmeta無効化 → 公開bootのFastboot書き込み → userdata消去 → 通常起動」を確認済みです。vbmeta無効化時の入力コマンド全文と書き込み後Flagsの読み戻し結果は報告に含まれておらず、上記はAOSPのFastboot実装に基づく操作例です。

参考: [AOSP Fastboot実装](https://android.googlesource.com/platform/system/core/+/master/fastboot/fastboot.cpp) / [AVBヘッダーとフラグ定義](https://android.googlesource.com/platform/external/avb/+/master/libavb/avb_vbmeta_image.h)

## Magisk適用済みbootの書き込み

```powershell
Get-FileHash .\boot-magisk-25.2-g06.img -Algorithm SHA256
# 上記SHA-256との一致を確認
```

書き込み先は `boot` です。`recovery`、MBOOT、userdataなどへ書き込まないでください。ブートローダーのFastboot書き込みが実際に使えることと対象パーティション名を確認できた環境でのみ、次の例を利用します。

```powershell
fastboot devices
fastboot flash boot .\boot-magisk-25.2-g06.img
# 転送・書き込みが両方OKAYの場合のみ
fastboot reboot
```

これはFastboot対応環境の操作例です。公開bootのFastboot書き込みと通常起動は、上記のユーザー実機検証で確認済みです。FAILEDが出た場合は中止してください。復旧時は利用者自身の対応する純正bootを使用します。

## 公開範囲・ライセンス

このReleaseにはMagisk適用済みbootと、同じバックアップから取得したvbmeta・vbmeta_systemを収録しています。vbmeta類は各1,048,576バイト、Flags=0で、改変せず添付しています。これらのvbmetaを書き込んだ状態での新たな起動試験は行っていません。全パーティションバックアップ、userdata、NVRAM、DRM鍵、端末の認証情報は収録しません。ramdiskのファイル一覧と設定ファイルを確認し、個人認証情報のファイルは見つかっていませんが、イメージ全体の安全性を保証するものではありません。

Magiskやカーネル等の権利はそれぞれの権利者に帰属します。イメージ全体に独自のオープンソースライセンスを付与するものではありません。出典とソース情報は [THIRD_PARTY.md](THIRD_PARTY.md) を参照してください。

## Support / 開発支援

開発・保守活動をKo-fiで支援できます。

[Ko-fiで支援する](https://ko-fi.com/long0823)
