"# repository_new"

--------------AdvPropの実装-------------------

講義の課題で実装したものです。

1. AdvPropとは
　AdvPropとは2019年11月21日にarXivで公開され、CVPR2020でも採用された論文 「Adversarial Examples Improve Image Recognition」(Cihang Xie and Mingxing Tan and Boqing Gong and Jiang Wang and Alan Yuille and Quoc V. Le, https://arxiv.org/abs/1911.09665)で提案された精度向上の手法です。
 
 2.論文の概要となにがすごいのか
 　これまで画像認識モデルの決定を誤らせる「adversarial examples（日本語では敵対的サンプルといわれることが多い）」に対してのロバスト性の向上しか考えられていなかったが、この論文では、それを画像認識モデルの精度向上に用いています。clean image（敵対的サンプルと比較する文脈で普通の画像のことを指す）と、そこから生成されたadversarial imageを同時に学習することで、ノイズを持った画像の分布を認識対象の真の分布から切り離し、認識精度の向上を達成しています。
  この手法のすごいとこは「学習中の工夫」で完結しているため、既存のモデルと組み合わせることでそのモデルの精度向上ができるとこです。論文中ではEfficientNetのb0~b7のすべてでこの手法による精度向上を示していました。2019年時点のSoTAがEfficientNet-b7なので必然的にSoTAを更新しました（b7では通常のImageNetのデータセットで0.7%の向上、b0では0.4%の向上）。
 
3. 実際の実装
  2でも若干言及しましたが、Cihang Xieらはadversarial imageの統計量への悪影響は分布が真の分布からずれることであると主張し、分布を正規化し調整するBatchNormalizationレイヤー（以下、BN）への改良を行いました。具体的には、補助BNを元のBNへ並列に追加し、補助BNへadversarial image、元のBNへはclean imageを学習時に通過させ、ほかのレイヤーはすべて共有させます。これにより補助BNのみがadversarial imageの分布を学習することになり、元のBNは真の分布のみ学習させることができます。こうして学習したモデルのclean imageの入出力で認識させることで、精度の向上を達成できます。

4. 実装のあれこれ
　SoTAならすでにコードが公開されているかもしれないと思いましたが、2020年8月3日時点ではPytorchが学習済みモデルを配布しているだけで、そのためのコードとかは特に見つかりませんでした（やること自体は簡単なせいかもしれませんが）。今回は論文でも用いていたEfficientNet-b0にAdvPropを実装しました（b7はgoogle colabの無料プランのメモリ量と作業量の関係で断念しました）。使用したデータセットはImageNetの一部分を配布しているImageNetteを用いました。
  注意点として現在のTensorflowではefficientnetはドキュメントには載っていますが、デフォルトでは入っていません。ドキュメントのページではTensorflow-2.3.0と表記され、実際に2.xをインストールすると2.2.0であることから、次のアプデでデフォルトで入るのかもしれません。
  また、論文で書かれているが実装できなかったのは「補助BNのパラメータ数は元のBNより0.5%程度多い」という点である（Tensorflow歴一週間なのでこれが限界です申し訳ありません）。
  
5. 結果
　このノートブックの設定で２回ほど実行しましたが、一回目は(88%,87%)、二回目は(92.99%, 92.15%)でAdvPropを実装したほうが高精度だった。最初は「増えた0.5%のパラメータがノイズ成分を効率よく学習する」ため精度が上がると思っていましたが、同サイズの補助BNでも精度の向上が確認できました。BN以外のレイヤーのパラメータは共有しているので、clean imageとadversarial image共通部分の学習のみが進むため、認識の解像度がピクセル単位から人間レベル（例えば、パンダなら白色と黒色で四足歩行していたり座って笹を食べていたり）になったためなんじゃないかと適当に思いました。
