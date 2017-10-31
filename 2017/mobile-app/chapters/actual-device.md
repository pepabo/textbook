# 実機との接続

Xcodeは手元の端末で動作検証が行えます。
大まかな手順としては、まずXcodeにAppleIDを登録し、そのIDとプロジェクトを紐付けます。その後iPhoneにアプリをインストールして、iPhoneの設定で自分のAppleID（開発元）を信頼します。

1. MacにiPhoneを接続
1. Xcodeで端末を選択
1. ⌘+R
1. プロジェクトとAppleIDが紐付いていないため"Failed to code sign"というエラーが出るので、"Fix Issue"を押す
1. AppleアカウントをXcodeに登録する
1. Personal Teamとして登録されるので、登録されたアカウントを選択する
1. 再度⌘+R
1. "Could not launch"というエラーが出る
1. iPhoneの設定から一般→デバイス管理と進み、登録したアカウントを信頼する
