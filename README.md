# TCL G06 / MT5867 Magisk適用済みbootイメージ

TCL 32S5400（G06 / MT5867）向け、Magisk **25.2（25200）** 適用済みのbootパーティションバックアップです。公式ファームウェアではありません。

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

**改変bootを書き込む前に、必ずvbmetaのAVB検証を無効化してください。** ブートローダーのアンロックとは別の操作です。対象TVと同じファームウェアのvbmetaを使い、元のboot・vbmeta等をバックアップして、復旧手段を確保してから進めます。

以下はブートローダーのFastboot書き込みに対応した環境での操作例です。今回の公開作業で、このFastboot書き込み経路を実機検証したものではありません。

```powershell
fastboot devices
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
# 転送・書き込みが両方OKAYであることを確認してからbootの書き込みへ進む
# FAILEDの場合は中止
```

Releaseのvbmeta類はFlags=0で、無効化済みではありません。別機種のvbmetaや空イメージを流用せず、vbmeta_systemも一律に書き換えないでください。MBOOTのmmc write.pにはFastbootのオプションは使えません。検証無効化は起動時の改変検知を弱めます。改変状態で再ロックしないでください。

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

これはFastboot対応環境の操作例です。このファイルについて、今回の公開作業でFastbootによる書き込み経路を検証したという意味ではありません。FAILEDが出た場合は中止してください。復旧時は利用者自身の対応する純正bootを使用します。

## 公開範囲・ライセンス

このReleaseにはMagisk適用済みbootと、同じバックアップから取得したvbmeta・vbmeta_systemを収録しています。vbmeta類は各1,048,576バイト、Flags=0で、改変せず添付しています。これらのvbmetaを書き込んだ状態での新たな起動試験は行っていません。全パーティションバックアップ、userdata、NVRAM、DRM鍵、端末の認証情報は収録しません。ramdiskのファイル一覧と設定ファイルを確認し、個人認証情報のファイルは見つかっていませんが、イメージ全体の安全性を保証するものではありません。

Magiskやカーネル等の権利はそれぞれの権利者に帰属します。イメージ全体に独自のオープンソースライセンスを付与するものではありません。出典とソース情報は [THIRD_PARTY.md](THIRD_PARTY.md) を参照してください。