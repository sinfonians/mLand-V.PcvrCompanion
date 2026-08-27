# mLand-V.PcvrCompanion

SteamVR/OpenVRが保持するHMD・コントローラー・トラッカーの現在姿勢と、VRCFaceTracking（VRCFT）の顔・目・口の表情を、同じPCのミューランドVへだけ渡すWindows用補助アプリです。VRM、音声、ルーム、アカウント情報は読み取りません。管理者権限、SteamVRドライバー追加、SteamVR設定変更、常駐サービスは不要です。

## 現在の公開状態

Windows x64自己完結版を試験提供します。ミューランドVの `/pcvr-companion` から、版を固定したGitHub ReleaseのZIPとSHA-256確認用ファイルを取得してください。SteamVR環境ごとの追従品質は引き続き実機検証中です。

## 実機確認手順

1. SteamVRを起動し、HMDと左右コントローラーがSteamVR上で認識されていることを確認します。
2. `Muland.PcvrCompanion.exe`を起動します。`mLand-V.PcvrCompanion`の状態画面が開きます。管理者権限は不要です。
3. GUIに表示された8桁コードを、ミューランドVの「PCVR（SteamVR）」欄へ入力します。localhost、port入力、ブラウザーのローカルネットワーク許可は不要です。期限切れの場合は、GUIの「接続コードを再発行」から新しいコードを発行します。
4. GUIでSteamVR、サイト接続、VRCFT、HMD・左右手・左右レーザー・最終姿勢送信の状態を確認します。HMDを正面へ向けて静止し、サイト側が`mLand-V.PcvrCompanion 自動設定: 3点`になるまで待ちます。
5. 腰・左右足のgeneric trackerが揃っている場合は、SteamVRのtracker roleを優先して6点へ自動設定します。GUIで6部位を一目で確認でき、roleなしtrackerは「6点候補」として表示されます。曖昧な場合は3点へ戻り、Web側の「トラッキング詳細設定」で間違っている部位だけ修正できます。
6. 終了時はGUIの`終了`またはウィンドウの閉じる操作を使います。終了するとOpenVR・VRCFT・LiveKit送信も停止します。

GUIの「VRスタジオ準備状況」は身体追従、表情、サイト接続の準備状態です。通常スタジオでは3点／6点の身体姿勢を既存の固定表示位置へ反映します。VRスタジオでは同じ身体姿勢と自由移動位置を分離し、配信用カメラをWeb視点とWeb立体音響の聴取点として同期します。接近制限と近距離カリングは既定OFFで、利用者がVR内設定から任意に有効化します。

### Perfect Sync表情（VRCFT）

1. 対応HMDの公式ランタイムとVRCFTを起動し、VRCFT側で機器モジュールがActiveになっていることを確認します。
2. Companionは起動時にVRCFT既定のloopback OSC出力9000を確保します。ミューランドVとの認証後にだけ、VRCFT入力9001へ自動設定通知を送ります。手動でVRCFTのportを変更しないでください。
3. 画面に`VRCFT受信中（目ON・表情ON・口ON）`等が出れば、顔はVRCFT、身体はOpenVR／WebXRが担当します。
4. VRCFT未検出時は音声リップシンクへ戻ります。OSC 9000をVRChat等が使用中の場合、顔受信だけ停止し、身体追従は継続します。

VIVE・Quest Pro系を対象とし、PICO 4 Pro／Enterprise系はVRCFTモジュール側の制約があるため試験中です。Quest単体VRは今回のVRCFT経路を使用しません。`forceRelevant`はVRCFTのglobal設定で所有権tokenがないため、Companion終了時に自動解除しません。顔受信を一度開始した後は、VRCFTを再起動するまで全表情の評価が続く場合があります。

Quest単体VRはこの補助アプリを使用しません。PCVR実機検収前のため、現在は3点追従と6点入力・脚IKを開発中として扱います。WebXRがコントローラー姿勢を返さないPCVR環境では、Companionが取得したOpenVR姿勢とトリガー・グリップ入力を腕、レーザー、VR内ボード操作、表情へ補完します。

## 安全境界

- CompanionからミューランドV APIとLiveKitへHTTPS/WSSの外向き接続だけを行います。公開ページからlocalhostへ接続しません。
- 8桁コードは短時間の一回限りです。認証後のLiveKit tokenと送信先はCompanionのメモリ内だけで扱い、コマンドライン、環境変数、ログ、永続Storageへ保存しません。
- 姿勢packetは認証した本人へだけLiveKit unicastし、claim済みsession UUIDをpacketへ結合して別sessionの再送・混入を拒否します。
- 端末installation secretはWindowsユーザーのDPAPIで暗号化して保存し、サーバーにはhashだけを保存します。
- 姿勢値、token、アカウント情報をファイルやログへ保存しません。
- 顔係数、VRCFTの生OSC packet、機器名・module名も保存・ログ表示しません。
- トラッカーの生serialはWebへ送りません。端末内の部分修正照合には、Companionで生成した匿名キーだけを使います。
- SteamVRのtracker roleは読み取り専用で参照し、設定ファイルへ書き戻しません。
- 終了するとLiveKit送信は直ちに停止します。サーバー側の短寿命tokenも再利用できません。

## 開発ビルド

`Bootstrap-OpenVr.ps1` は公式OpenVR SDK v2.15.6、`Bootstrap-LiveKit.ps1` は公式LiveKit C++ SDK v1.6.0を取得し、どちらも固定SHA-256で検証します。`build.ps1` はネイティブ送信ブリッジとリポジトリ内のローカル.NET 8 SDKを使用し、システム全体を変更しません。

`build.ps1 -Publish`でwin-x64自己完結版を生成します。GitHub Releaseへは、サイトと同じcommitから生成し、EXEのProductVersion、ZIP内容、SHA-256、self-testを確認した成果物だけを掲載します。
