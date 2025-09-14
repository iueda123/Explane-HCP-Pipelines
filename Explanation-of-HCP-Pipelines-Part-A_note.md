# 小池研勉強会資料：HCP Pipelines の基本的説明

2025.09.05 Issei Ueda

--------

## この資料の内容

* **Human Connectome Project (HCP) とは**
* **HCP Pipelines とは**
  * 概要
  * 基本情報
  * 関連リソース
  * 主な特徴
  * 代表的なモジュール
  * A tip: 推奨される表記について
* **HCP Pipelines を使うことのメリット**
  * Voxel-Based (VBA) vs. Surface-Based (SBA) アプローチ
  * FreeSurfer vs. MSMレジストレーション
* **最近のバージョンアップに伴う変更点**
  * 主要バージョンリリース
  * 詳細情報
* **HCP Pipelines を走らせるための環境構築**
* **HCP Pipelines で脳を適切に処理するために必要な画像/情報**
  * Structual MRI (sMRI)
  * Functional MRI (fMRI)
  * Diffusion MRI (dMRI)
  * Tips
* **具体的な画像の例**
* **必要となるプロトコル、画像について具体例**
  * HCP-YAの場合
* **HCP Pipelines Overview**
  * 01. PreFS (Pre-Free-Surfer) Pipeline
  * 02. FS (FreeSurfer) Pipeline
  * 03. PostFS (Post-Free-Surfer) Pipeline
  * 04. GenericfMRIVolume（fMRIボリューム処理）
  * 05. GenericfMRISurface（fMRI表面処理）
  * 06. IcaFix（ICA+FIX）
  * 07. PostFix（Post-FIX処理）
  * 08. MSMAll（マルチモーダル表面マッチング）
  * 09. DeDriftAndResample（ドリフト補正・リサンプリング）
  * 10. RestingStateStats（安静時fMRI統計）
  * 11. Diffusion Preprocessing Pipeline

--------

## Human Connectome Project (HCP) とは

* 2009年に開始された米国国立衛生研究所（NIH）が後援するヒトの脳内の解剖学的及び機能的結合性であるコネクトームを解明することを主眼としたプロジェクト。​
  * 米国のConnectome Coordination Facility (CCF) という研究支援組織によって進められている。​
  * ワシントン大学、ミネソタ大学を中心に、オックスフォード大学、マサチューセッツ工科大学、ハーバード大学、UCLA などが参画。​
* ヒトの脳内の結合情報に着目した複数のプロジェク群からなる。​
  * 健常成人のプロジェクト (HCP-YA)​
  * 脳の発達に焦点を当てたプロジェクト(HCP-D)、脳の老化に焦点を当てたプロジェクト（HCP-A)​
  * 脳疾患を対象としたプロジェクト（CRHD)​
  * 脳の結合性解析用ソフトウェアに関するプロジェクト 等​
* Elam J S, et al., Neuroimage, 2021 (PMID: 34508893)​にプロジェクトの詳細、成果がよくまとめられている。​
* プロジェクトで蓄積された生データや加工データの多くは広く無償で公開されている。
* 国際脳ヒト脳MRIプロジェクト（BMB-HBM）など、多くのプロジェクトがHCPの撮像法・処理方法を参考にしている（HCP-Sytle）

![HCP Connectome Coordination Facility Homepage](img/HCP_Homepage.png)
*図1: HCP Connectome Coordination Facility Homepage - HCPプロジェクトの統括組織であるCCFのWebサイト。Young Adult HCP、Lifespan HCP、病気関連プロジェクト、HCPソフトウェアなどの情報が集約されている。*
(https://www.humanconnectome.org/)

--------

## HCP Pipelines とは

### 概要

  * HCP Pipelinesとは、HCPで開発された**脳MRI画像処理の標準化パイプライン群**。
  * HCP-Style脳画像前処理スクリプト集として、複数被験者・複数施設間でも一貫した高精度な脳画像解析を可能にする。  
  * コンセンサスの得られている適切なバイアス除去やレジストレーションが行われ、その後の処理がやりやすくなる。


### 基本情報

  * **開発・配布**: GitHub上に無償公開
  * **対象**: HARPやCRHDなどのHCP-Styleで撮像された画像
  * **構成**: シェルスクリプトやPythonスクリプトで構成され、FSL、FreeSurfer、Workbench などの外部ツールを組み合わせて動作
  * **留意点**: あくまでも前処理スクリプト集であり、これに通すだけですぐに研究に使える数値が得られるわけではない
  * **導入**: 手に入れるのは簡単だが、セッティングにはコツや慣れが要る

### 関連リソース

  * [Documentation and Release Notes, Frequently Asked Questions](https://www.humanconnectome.org/software/hcp-mr-pipelines)
  * [Google Group](https://groups.google.com/a/humanconnectome.org/g/hcp-users?pli=1) - 非常に活発
  * [GitHub](https://github.com/Washington-University/HCPpipelines) - 非常に活発

### 主な特徴

**マルチモーダル対応:** 
構造MRI（T1w, T2w）、拡散MRI（dMRI）、機能MRI（fMRI）を対象とし、全体を通して統一された処理フレームワークを提供する。

**表面ベース解析 (Surface-based analysis):** 
脳皮質の幾何学的特徴を考慮し、皮質表面上でデータを整列させるため、個人差をより正確に補正できる。

**再現性重視:** 
画像取得から前処理、標準空間へのマッピングまでを自動化し、他の研究者が同じデータを同じ手順で処理できるよう設計されている。

### 代表的なモジュール

* **PreFreeSurferPipeline.sh**: B0歪みやバイアス場補正などの前処理
* **FreeSurferPipeline.sh**: FreeSurferを利用した皮質再構築
* **PostFreeSurferPipeline.sh**: FreeSurfer出力をHCP標準表面空間に変換
* **fMRIVolume/fMRISurfacePipeline.sh**: fMRIデータのボリュームおよび表面ベースの前処理
* **DiffusionPipeline.sh**: 拡散MRIの前処理と接続性解析準備

![HCP Pipeline Scripts Directory Structure](img/Inside_of_HCPpipelines.png)
*図2: HCP Pipelines スクリプトディレクトリ構造 - HCPpipelinesの主要なスクリプト群。Examples/Scripts/フォルダには実行用スクリプトが含まれ、各パイプライン（PreFreeSurfer、FreeSurfer、PostFreeSurfer、fMRIVolume、fMRISurface、Diffusion等）に対応している。黄色矢印は主要な6つのMinimal Preprocessing Pipelinesを示す。*

### A tip: 推奨される表記について

  * 文章中での正式表記: HCP Pipelines
  * 変数名、ファイル・フォルダ名: HCPpipelines

公式GitHubリポジトリの説明では "Human Connectome Project Pipelines" という表現が使われているが、略称として "HCP Pipelines" も使われている。論文や公式マニュアルでも "HCP Pipelines" と表記されている。リポジトリ名としては "HCPpiplines" という形（大文字HCP＋小文字pipelines、スペースなし）も使われている。略記・単数形（HCPpipeline, HCPPipeline等）は公式には使われていないため避けた方が良い。



--------
## HCP Pipelines を使うことのメリット

HCP Pipelinesを利用することで、脳画像解析において以下のような多くのメリットが得られる。

1.  **高精度かつ標準化された前処理**:
    *   構造MRI (sMRI)、機能MRI (fMRI)、拡散MRI (dMRI) にわたり、現在コンセンサスが得られている適切な手法を組み合わせた前処理を統一的に実行できる。
    *   Gradient-echo/Spin-echo field mapを用いた歪み補正、Eddyカレント補正など、複数の高度な補正技術が統合されている。

2.  **Surface-Based Analysis (SBA) による精緻な解析**:
    *   従来のVoxel-Based Analysis (VBA) と異なり、脳を2次元の皮質表面として扱うことで、個人間の脳の形状の違いをより正確に補正する。
    *   これにより、神経生物学的により妥当性が高く、被験者間の微妙な解剖学的・機能的差異を保持したまま比較することが可能になる。

3.  **MSMレジストレーションによる高度な位置合わせ**:
    *   FreeSurferが脳溝・脳回のみを基準にするのに対し、HCP Pipelinesで採用されているMSM (Multi-modal Surface Matching) は、脳溝・脳回に加えてミエリンマップや機能的ネットワークといった複数の情報を利用する。
    *   これにより、FreeSurferでは困難だった機能的に等価な領域の対応付け精度が向上し、より生物学的に意味のある位置合わせが実現される。

4.  **大規模公開データとの互換性**:
    *   HCP-YA (Young Adult), HCP-A (Aging), HCP-D (Development) といった大規模な公開データセットと同じ手法でデータ処理を行うため、自施設のデータをこれらのデータセットと直接比較・統合した解析が容易になる。

### Voxel-Based (VBA) vs. Surface-Based (SBA) アプローチ

HCP特徴であるSBAについてVBAと比較してその有用性を確認したいと思う。

| 特徴 | Voxel-Based Analysis (VBA) | Surface-Based Analysis (SBA) |
| :--- | :--- | :--- |
| **基本単位** | 3次元のボクセル（立方体） | 2次元の皮質表面 |
| **位置合わせ** | 空間的な平滑化（スムージング）を多用 | 脳の折り畳み構造に基づき、平滑化を最小限に抑制 |
| **個人差の扱い** | 平均化され、失われやすい | 保持されやすく、詳細な比較が可能 |
| **精度** | 領域境界が不正確になりがち | 皮質領域の境界をより正確に特定可能 |

![SBA vs VBA Uncertainty Comparison](img/sba_vba_comparison.png)
*図: SBAとVBAの精度比較 (Coalson et al, PNAS, 2018)。SBA（上段）はVBA（下段）に比べ、皮質領域境界の不確実性（黄色や赤色）が低く、より正確な領域特定が可能であることを示している。*

<!--
/home/iu/Dropbox/025_TokyoUniv/TokyoUniv.mm.files/Coalson2018.pdf
-->

### FreeSurfer vs. MSMレジストレーション

SBAで最も代表的なソフトウェアはFreeSurferで、
HCP PipelinesもFreeSurferを活用してSBAを実現していますが、
HCP Pipelinesは、SBAの精度をさらに高めるために、
FreeSurferを使ったレジストレーションに加えて、MSMレジストレーションも採用し精度を高めている。

| 特徴 | FreeSurfer（従来法） | MSM (HCP Pipelinesで採用) |
| :--- | :--- | :--- |
| **基準情報** | 脳の形状（脳溝・脳回）のみ | **マルチモーダル**: 形状＋機能＋ミエリンなど |
| **位置合わせ** | 大局的な形状パターンを合わせる | 生物学的に関連のある領域同士を対応付ける |
| **結果** | 個人差が平滑化されやすい | 局所的な構造・機能の個人差が保持されやすい |

![MSM Registration Optimization](img/msm_registration_concept.PNG)
*図4: MSMレジストレーション最適化の概念図 (Robinson et al., 2014)。複数の情報を用いて皮質の対応関係を最適化し、より精密な位置合わせを実現する。*

MSMの利点:
  * 局所的な 構造、機能の対応をより精緻に取得
  * FreeSurferで「テンプレートに吸収」される細かい形態の違いをより保存
  * MSMSulc + MSMAll により局所的な構造‐機能対応がより正確に整列
  * 個人差が消えすぎず、局所的特徴が残りやすい

理論的には、MSMで得られる特徴量の方が、FreeSurferに比べて局所的な個人差を多く含むと考えられる。


このように、HCP PipelinesはSBAとMSMを組み合わせることで、従来の解析手法の限界を克服し、より高精度で信頼性の高い脳画像解析を可能にする。

--------

## 最近のバージョンアップに伴う変更点

HCP Pipelinesは活発に開発が続けられており、近年も重要なアップデートがリリースされている。

### 主要バージョンリリース

#### Version 5.0.0 (2024)
- HCP Pipelinesが4.xから5.xへメジャーバージョンアップ
- 主要な新機能追加により大幅なバージョン番号変更
- 4.xツリーとの後方互換性を維持

#### Version 4.8.0 (2023)
- Adult Aging Brain Connectomes (AABC, HCP-Aging follow-on study) の処理に使用
- 非HCP/非Siemens T1w、T2w、T2w-FLAIR画像への対応強化
- 新しい抽象化されたミエリンマップBCモジュールの追加

#### Version 4.3.0 (2023)
- HCP Aging、HCP Development、Connectomes Related to Human Disease データの処理に使用
- 構造QCおよびfMRI QC用のWorkbenchシーン生成機能を追加
- Philips フィールドマップを使用した歪み補正オプションの追加
- DiffPreproc に `--select-best-b0` オプションを追加

#### Version 4.1.3 (2023)
- HCP Lifespan Aging and Development データの機能前処理に使用
- 非HCP-Styleレガシー MRI データの処理サポートを追加
- `--processing-mode` パラメータによるレガシーデータ処理モードの追加

### 詳細情報
- **公式GitHub**: https://github.com/Washington-University/HCPpipelines
- **リリースノート**: https://github.com/Washington-University/HCPpipelines/releases


--------


## HCP Pipelines を走らせるための環境構築

HCP PipelinesはGitHub上の公開され、活発に整備がなされているが、
様々なツールを連携して動くものとなっているため、
適切に動く環境を構築するのに手間がかかる。
インストール先OSの種類やバージョン、
連携させるツールのバージョンによって細かな調整が必要となる。

**HCP Pipelinesを走らせるために必要な主なツール**

- **HCPpipelines**: 本体 (GitHub)
- **MATLAB Runtime**: コンパイル済みMATLABコードの実行環境
- **FreeSurfer**: 皮質再構成ツール (ver. 6推奨)
- **FSL**: FMRIB Software Library (FIXなどに必要)
- **Connectome Workbench**: 可視化・解析ツール
- **gradunwarp**: 勾配歪み補正ツール

**備考**
  * FreeSurfer ver. 6 推奨
  * FSL FIX関連のインストール部分が難所。Rのバージョンやライブラリの制約が厳しい。
  * 根本先生が https://www.nemotos.net/?p=3613 にインストール方法を公開している。
  * 根本先生が公開している Lin4Neuro にHCPpipelinesが組み込まれている。
  * 上田は [Ubuntu 20.04/22.04 上に HCPpipelines release 4.3.0/4.5.0 をインストールするノウハウ](http://isue.dix.asia:8890/dokuwiki/das/doku.php?id=hcppipelines:how_to_setup_hcp_pipelines_environment)、[HCPpipelines release 4.3.0 on Ubuntu 18.04 with Docker Image作成方法](http://isue.dix.asia:8890/dokuwiki/das/doku.php?id=hcppipelines:deploy_docker_environment_of_hcppipelines_v4.3.0_on_ubuntu_18.04)を持っているが整理しきれていない。困られている方がいらっしゃればご相談ください。
  
-------

## HCP Pipelines で脳を適切に処理するために必要な画像/情報

撮像プロトコルについての説明は 2024.09.06 に理化学研究所で説明させていただいたときの資料を参照のこと。

ここでは、HCP Pipelinesで取り扱う３種の主要なMRI画像に推奨される条件を確認してゆく。

### Structual MRI (sMRI)

  * T1-weightd image: 全頭頭部画像、0.7–1.0 mm iso 推奨。
  * T2-weighted image: 必須ではないが、強く推奨される。皮質、灰白質の弁別精度が上がる。解像度が T1w と同等のもの。
  * Gradient distortion 補正係数: メーカー配布の *.grad ファイル。無い場合は未補正でも動くが精度が低下。
  * コツとして、T1w、T2w は頭皮までカバーされていると Bias補正や頭蓋骨を外すためのマスク作成が安定。
  
### Functional MRI (fMRI)

  * BOLD EPI 本体
    * 少なくとも安静時画像を用意することが推奨
    * 各種タスク時のものがあればそれらも処理に加えられる。
    * 解像度は施設の SNR と時間で決定（一般的に fMRI 2.0–2.5 mm）。
    * TR/MB（fMRI）：パイプライン自体は固定要件なし。短 TR（例 0.7–1.0 s）は有利。
  * Spine Echo Field Map (SEFM): 
    * 逆位相エンコードのペア（AP/PA もしくは LR/RL）の Spin-Echo EPI（SE-EPI） を各セッションで取得
      * B0 歪み補正等に使う。
    * SE-EPI ペアの readout 時間や PE 方向は本画像のものと一致させる
    * SE-EPI ペアが無い場合、gradient echo の field map（位相差法）を代替として選択できる
  * Single Band Reference Image (SBRef)
    * SBRefはmultibandで取得した画像の処理精度向上に有用


### Diffusion MRI (dMRI)

  * DWI EPI 本体
    * b=0 を含む。
    * 解像度は施設の SNR と時間で決定（一般的に DWI 1.5–2.0 mm iso）。
    * マルチシェル推奨（例：b≈1000/2000/3000、各シェルで十分な方向数。HCP では各 90 方向程度だが、研究環境では各 30–60 方向でも可）
  * bvals / bvecs 情報
  * Single Band Reference Image (SBRef)
  * 逆位相エンコードの b=0 ペア（AP/PA 等の blip-up/blip-down）：TOPUP 用。
  

### Tips

**HCP-YAにあるMagnitude画像、Phase画像は何に使う？**

HCP-YAデータセットにMagnitude画像、Phase画像がある理由は、古典的なEPI歪み補正方法も試せるようにしているためであると思われる。fMRI処理やdMRI処理においてEPIによる歪み補正にField Mapが使われるが、その生成方法としてHCP Pipelinesでは次の２種類をサポートしている。

1. Gradient-Echo Field Map を利用した方法

  * Phase/Magnitude由来のField Mapを利用
  * このようなField Mapをつかった補正方法は「位相差法：と呼ばれる。先発。古典的。
  * 信頼性に限界あり（頭蓋底や鼻腔周囲で不安定）
    
2. Spin-Echo Field Map を利用した方法

  * 逆位相PEのSpin-Echo EPIペア（AP/PAまたはLR/RLの反転位相方向で取得）由来のField Mapを利用
  * このようなField Mapをつかった補正方法は「TOPUP法」と呼ばれる。後発。推奨。
  * より頑健にEPI歪みを補正可能    

HCP-YA（HCP-A、HCP-Dも？）の公開データセットには「Gradient-Echoの Magnitude/Phase 画像」と「Spin-Echo逆位相EPIペア」の両方が含まれている。したがって、GE Field Mapによる古典的な補正も試せる。ただしHCPで推奨されている補正方法はSpin-EchoペアによるTOPUP法で、HCP-YA, HCP-A, HCP-D データセットではこちらによる処理が行われている。


**SBRefはmultibandで取得した画像の処理精度向上に有用**
  * Single Band Reference Image (SBRef) は、fMRI/dMRI HCP-Style撮像の際に取得される補助的な画像。
  * fMRI処理におけるモーション補正や空間整合性の参照画像、dMRI処理におけるエディカレント補正やb0整列のための参照画像として使う。  
  * fMRI/dMRI の本撮像では  multiband で同時に複数スライスを取得するが、SBRef撮像では通常通りの1スライスずつ取得する。そのためSBRef は信号雑音比（SNR）が良く、歪みが少ない。歪みの強い 本撮像のfMRI/dMRI をそのままT1に合わせるよりも、歪みの少ない SBRef リファレンス画像を介して解析したほうが、計測精度が上がる。

**SEFMはfMRI用、dMRI用を別々に用意すべき**

HCP-Style撮像では、 位相エンコード（AP/PA もしくは LR/RL）で 同一ジオメトリ・同一読み出し帯域の SE-EPI を取得する。このとき fMRI 用と DWI 用で別々に取得すべきとされている。

SEFMをfMRI用、dMRI用を別々に用意すべき理由は、①本撮影の直前に撮ったほうが高い精度でField Map推定ができるため、②それぞれの撮影条件に合わせたField Mapを撮影すべきだからである。

B0場の不均一性（主に空気と組織の境界で発生）は、スキャナの温度変化、被験者の体動や体位の微妙な変化により、時間とともに変化する。そのため、本撮影の直前に field map を取ることで、より正確な補正が可能になる。

またfMRIとdMRIは撮像条件が異なり、位相エンコード方向やEPI readoutの違いによって、同じ被験者・同じ時間でも歪みパターンが異なる可能性がある。よって 撮像モードに合わせた SEFM を使う必要がある。

**dMRI において dir95, dir96, dir97 と軸数が異なっている理由**

各ブロックの方向数が微妙に異なるのは、勾配方向の最適化と品質管理上の理由で、あらかじめ設計された仕様。
各被験者について 3シェル (b=1000, 2000, 3000)
各シェルは 90方向 + いくつかのb0
結果的に 288方向の画像となっている。

**最小セット**
  * T1w（1 mm 等方、全頭）
  * fMRI：各 run の BOLD と SE-EPI の逆位相ペア
  * DWI：b=0 を含む DWI、blip-up/blip-down の b=0、bvals/bvecs

**Session, Run, Block という表現について**

  * Session: 被験者がMRI装置に入ってから出るまでの、一連の撮像のまとまり
  * Run: セッション内で行われる 1回の連続したfMRI撮像 の単位
  * Block: Sessionよりも小さく、Runよりも大きな塊で撮影のまとまり

HCP-YAでは 1 session あたり fMRI が 4 runs ある。それら 4 runs は day 1に取得された block、day 2 に撮影された block に分かれる。また それら4 runsのうち 2 runs は LR block であり、2 runs は RL block である。 

<!--

ToDo: sMRI、fMRI、dMRIを適切に処理するのに必要な撮像メタ情報を整理してください。
    * PhaseEncodingDirection, TotalReadoutTime/EffectiveEchoSpacing の情報が必要（TOPUP用）。
    * BIDS の JSON に PhaseEncodingDirection, TotalReadoutTime（または EffectiveEchoSpacing と AcquisitionMatrixPE）を正確に。TOPUP/EDDY で必須。？


**撮像マシンメーカーについて**

同程度の補正効果が得られるかは別として、走らせることは可能。

**駒場の場合(iTTC-5th, HARP)**

BIDS 構成例
sub-01/anat/sub-01_T1w.nii.gz, sub-01_T1w.json
sub-01/anat/sub-01_T2w.nii.gz, sub-01_T2w.json
sub-01/func/sub-01_task-rest_dir-AP_bold.nii.gz と ..._dir-PA_bold.nii.gz、*_sbref.nii.gz（任意）、各 JSON
sub-01/fmap/sub-01_dir-AP_run-01_epi.nii.gz, sub-01_dir-PA_run-01_epi.nii.gz（SE-EPI フィールドマップ）
sub-01/dwi/sub-01_dwi.nii.gz, sub-01_dwi.bvec, sub-01_dwi.bval, および fmap の SE-EPI ペア


https://chatgpt.com/c/68b694f8-5388-832a-bc3c-4c3f0227aa72
-->

--------

## 具体的なデータセットの例：HCP-YA

HCP Pipelinesで処理するデータセットは、Connectome DBから入手できます。アカウントを作成すれば、特別な申請なしにHCP-YA（Young Adult）などのサンプルデータをダウンロード可能である。

![ConnectomeDB Public Data Access](img/ConnectomeDB.jpg)
*図5: ConnectomeDB Public Data Access Interface - HCPデータベースの公開データ取得画面。*

HCP-YAデータセット（例：被験者100307）には、主に以下の3種類の画像データが含まれている。

### 1. 構造MRI (sMRI)
高解像度の解剖学的情報を提供します。
*   **T1w画像** (`*_T1w_MPR1.nii.gz`)
*   **T2w画像** (`*_T2w_SPC1.nii.gz`)
*   **磁場マップ画像 (FieldMap)**: 歪み補正に使用します (`*_FieldMap_Magnitude.nii.gz`, `*_FieldMap_Phase.nii.gz`)。

### 2. 機能MRI (fMRI)
安静時やタスク時の脳活動を記録します。データは通常、位相エンコード方向が異なるペア（RL/LR）で提供される。
*   **BOLD画像**: 脳活動の時系列データ (`*_rfMRI_REST1_RL.nii.gz`)
*   **SBRef画像 (Single-Band Reference)**: BOLD画像の歪みが少なく、位置合わせの基準として使用 (`*_rfMRI_REST1_RL_SBRef.nii.gz`)
*   **スピンエコー磁場マップ (SpinEchoFieldMap)**: BOLD画像の歪み補正に使用する、逆位相エンコードのペア (`*_SpinEchoFieldMap_RL.nii.gz`, `*_SpinEchoFieldMap_LR.nii.gz`)。

### 3. 拡散MRI (dMRI)
脳の白質線維の走行を可視化します。複数の拡散方向 (dir95, dir96, dir97など) で撮像される。
*   **DWI画像**: 拡散強調画像 (`*_DWI_dir95_RL.nii.gz`)
*   **bval/bvecファイル**: 拡散勾配の強度と方向を示す情報 (`*_DWI_dir95_RL.bval`, `*_DWI_dir95_RL.bvec`)
*   **SBRef画像**: DWIの位置合わせの基準として使用 (`*_DWI_dir95_RL_SBRef.nii.gz`)

このように、各モダリティの画像本体に加え、歪み補正や位置合わせ精度を向上させるための補助的な画像・情報がセットになっている。


### HCP-YAの場合
  * sMRI
    * 100307_3T_T2w_SPC1.nii.gz
    * 100307_3T_T1w_MPR1.nii.gz
    * 100307_3T_FieldMap_Magnitude.nii.gz
    * 100307_3T_FieldMap_Phase.nii.gz
  * fMRI
    * rfMRI_REST1
      * 100307_3T_SpinEchoFieldMap_RL.nii.gz
      * 100307_3T_SpinEchoFieldMap_LR.nii.gz
      * 100307_3T_rfMRI_REST1_RL_SBRef.nii.gz
      * 100307_3T_rfMRI_REST1_RL.nii.gz 
      * 100307_3T_rfMRI_REST1_LR_SBRef.nii.gz
      * 100307_3T_rfMRI_REST1_LR.nii.gz
    * rfMRI_REST2
      * 100307_3T_SpinEchoFieldMap_RL.nii.gz
      * 100307_3T_SpinEchoFieldMap_LR.nii.gz
      * 100307_3T_rfMRI_REST2_RL_SBRef.nii.gz
      * 100307_3T_rfMRI_REST2_RL.nii.gz 
      * 100307_3T_rfMRI_REST2_LR_SBRef.nii.gz
      * 100307_3T_rfMRI_REST2_LR.nii.gz
  * dMRI
    * dir95
      * 100307_3T_DWI_dir95_RL_SBRef.nii.gz
      * 100307_3T_DWI_dir95_RL.nii.gz
      * 100307_3T_DWI_dir95_RL.bval
      * 100307_3T_DWI_dir95_RL.bvec
      * 100307_3T_DWI_dir95_LR_SBRef.nii.gz
      * 100307_3T_DWI_dir95_LR.nii.gz
      * 100307_3T_DWI_dir95_LR.bval
      * 100307_3T_DWI_dir95_LR.bvec
    * dir96
      * 100307_3T_DWI_dir96_RL_SBRef.nii.gz
      * 100307_3T_DWI_dir96_RL.nii.gz
      * 100307_3T_DWI_dir96_RL.bval
      * 100307_3T_DWI_dir96_RL.bvec
      * 100307_3T_DWI_dir96_LR_SBRef.nii.gz
      * 100307_3T_DWI_dir96_LR.nii.gz
      * 100307_3T_DWI_dir96_LR.bval
      * 100307_3T_DWI_dir96_LR.bvec
    * dir97
      * 100307_3T_DWI_dir97_RL_SBRef.nii.gz
      * 100307_3T_DWI_dir97_RL.nii.gz
      * 100307_3T_DWI_dir97_RL.bval
      * 100307_3T_DWI_dir97_RL.bvec
      * 100307_3T_DWI_dir97_LR_SBRef.nii.gz
      * 100307_3T_DWI_dir97_LR.nii.gz
      * 100307_3T_DWI_dir97_LR.bval
      * 100307_3T_DWI_dir97_LR.bvec

※SEFM は REST1_RLデータパッケージ と REST1_LR データパッケージ それぞれに含まれるが、同一画像である。
 

------

## Each HCP Pipeline

次に、HCP Pipelinesに含まれるそれぞれのPipelineを取り上げ、
HCP-Styleと呼ばれる撮像法で収集された画像にどの様な処理が施されてゆくかの理解を深めたいと思う。

（国際脳プロトコルは、Human Connectome Project のプロトコルに合わせ、 
高性能MRI機器を用いてマルチモダリティに構造画像、機能画像を短時間で取得できるように開発されている。）

HCPpipelinesのコアな部分は HCP Minimal Preprocessing Pipelines と呼ばれ、Glasser, Neuroimage, 2013の論文に詳細が説明されている。
HCP Minimal Preprocessing Pipelines は６本からなる。
1. PreFreeSurfer Pipeline, 2. FreeSurfer Pipeline, 3. PostFreeSurfer Pipeline, 4. fMRIVolume Pipeline, 5. fMRISurface Pipeline, 6. Diffusion Preprocessing Pipeline
である。

これらパイプラインを走らせるスクリプトは、HCPpipelinesフォルダの下のScriptsフォルダの直下に存在する。
走らせる順番については決まり事があり、
PreFreeSurfer Pipeline、FreeSurfer Pipeline、PostFreeSurfer Pipelineをこの順で走らせた後、
Functional MRI前処理のために、fMRIVolume Pipeline、fMRISurface Pipelineを走らせ、
Diffusion MRI前処理のために、Diffusion Preprocessing Pipelineを走らせる。

さらにfMRIについては ExtraのPreprocessing Pipelinesが用意されていて、
Glasserらの2016年の論文に従うならば、IcaFix、PostFix、MSMAll、DeDriftAndResample、RestingStateStatsといった追加のパイプラインを走らせる必要がある。

![HCP Minimal Preprocessing Pipelines Overview](img/glasser2013_fig07.png)
*図6: HCP Minimal Preprocessing Pipelines 全体概要図 (Glasser et al., 2013) - HCP構造パイプライン（PreFreeSurfer、FreeSurfer、PostFreeSurfer）を順次実行し、その後HCP機能パイプライン（fMRIVolume、fMRISurface）またはDiffusion Preprocessing Pipelineを実行する。さらにfMRI Denoising and Analysis、dMRI Denoising and Analysis、Diffusion Analysisへと続く。*

HCPpipelines/Examples/Scripts/ に例がある。


### 01. PreFS (Pre-FreeSurfer) Pipeline

HCPpipelines１本目のパイプラインは Pre-FreeSurfer Pipelineである。
このパイプラインはPreFreeSurferPipelineBatch.sh というスクリプトによって呼び出される。 
このパイプラインではGradient Distortion Correction, Readout Distortion Correction, B1−やB1+ bias field Correction等によるバイアスの補正、
T1強調画像とT2強調画像の重ね合わせ、
被験者空間データのMNI空間への合わせ込みが行われる。
このパイプラインの出力は主に T1wフォルダ直下、MNINonLinearフォルダ直下へ行われる。

**目的**: FreeSurfer処理前の構造画像前処理

**主要処理**:
- T1w/T2w画像の読み込みとACPC整列
- 磁場歪み補正（Gradient Echo、Spin Echo対応）
- MNI152標準空間への非線形レジストレーション
- Brain extraction

**入力**:
- `sub-*/anat/*_T1WI.nii.gz`（必須）
- `sub-*/anat/*_T2WI.nii.gz`（オプション）
- `sub-*/fmap/*_SEFM-Mgntd_*.nii.gz`（SEF対応時）

**主要パラメータ**:
- 脳サイズ: 150mm（人間標準）
- エコースペーシング: 0.00058s
- 勾配歪み係数: SimensPrismaKomaba_coeff.grad



![PreFreeSurfer Pipeline Workflow](img/glasser2013_fig09.png)
*図7: PreFreeSurfer Pipeline処理フロー (Glasser et al., 2013) - T1w/T2w画像の歪み補正から始まり、ACPC整列、脳抽出、Readout歪み補正、T1w（Native Volume Space）とT2w（undistorted）を経て、BBR Cross-modalレジストレーション、Bias Field補正、MNI Nonlinear Volume Registrationまでの処理流れを示す。*


### 02. FS (FreeSurfer) Pipeline

HCPpipelines２本目のパイプラインは FreeSurfer Pipeline である。
FreeSurfer Pipeline は Structural MRI 画像処理のための２番目のPipelineになる。
このPipelineを走らせるためのスクリプト名は「FreeSurferPipelineBatch.sh」である。
このPipelineでは、次のような事が行われる。
FreeSurfer のrecon-all処理をベースに、
大まかな脳領域抽出、白質境界、T2wを用いた軟膜境界の正確な推定、
脳表面のテッセレーション、球体膨張、fsaverageへの位置合わせ、
脳の折りたたみ構造に基づく領域特定、皮質下灰白質の抽出 である。

このパイプラインの主な出力先は T1wフォルダ下の被験者IDと同じ名前をもつフォルダである。

**目的**: 皮質表面再構成と皮質下構造分割

**主要処理**:
- 皮質表面メッシュ生成（白質・軟膜表面）
- 皮質厚測定
- 皮質下構造セグメンテーション

**入力**:
- `T1w_acpc_dc_restore.nii.gz`
- `T1w_acpc_dc_restore_brain.nii.gz` 
- `T2w_acpc_dc_restore.nii.gz`

**特記事項**:
- シングルコアモード設定（NSLOTS=1）
- HCPStyleData処理モード



![FreeSurfer Pipeline Workflow](img/glasser2013_fig12.png)
*図8: FreeSurfer Pipeline処理フロー (Glasser et al., 2013) - Bias補正されたNative Volume空間のT1w/T2w（0.7mm）から開始し、T1wを1mmにダウンサンプル後、recon-all autorecon1（Brain Mask Assist）とautorecon2（Initial White Surface）を実行、高解像度白質表面配置を行い、recon-all継続（Final White Surface〜Pial Surface）、高解像度軟膜表面配置を経て、recon-all完了（fsaverageへの表面レジストレーション含む）まで。*

### 03. PostFS (Post-FreeSurfer) Pipeline

HCPpipelines３本目のパイプラインは Post-FreeSurfer Pipeline である。
Structural MRI画像処理のための３番目のPipelineになる。
Post-FreeSurfer Pipeline は PostFreeSurferPipelineBatch.sh から呼び出すことができる。
このPipeline では、次のような事が行われる。
    • FreeSurferの出力ファイルをNIfTI, GIfTIフォーマットにする
    • 左右の大脳半球の位置の対称化と均等なメッシュへ再構成
    • 脳回情報に基づいた「Gentleな」皮質表面位置合わせを行う
    • ミエリンマップの作成 ・B1+バイアス補正
    • mid-thickness, Inflated、very inflated surfaces の生成
    • Specファイルの生成
である。

Post-FreeSurfer Pipelineの主な出力先は
  * T1wフォルダ下のNativeフォルダ
  * MNINonLinearフォルダ下のNativeフォルダ
  * MNINonLinearフォルダ直下
  * MNINonLinearフォルダ下のfsaverage_LR32kフォルダ
  * T1wフォルダ下のfsaverage_LR32kフォルダ
である。

**目的**: FreeSurfer結果の標準化と表面メトリクス計算

**主要処理**:
- 表面メッシュのfsaverage標準空間変換
- 皮質厚・曲率・ミエリン等のマッピング
- 32k_fs_LRメッシュへのリサンプリング



![PostFreeSurfer Pipeline Workflow](img/glasser2013_fig16.png)
*図9: PostFreeSurfer Pipeline処理フロー (Glasser et al., 2013) - FreeSurferのrecon-all出力をNIfTI/GIfTI/NIFTIに変換し、Final Brain Mask生成、Cortical Ribbon Volume生成、T1w/T2w皮質ミエリンマップ生成、ミエリンマップのConte69グループ平均への正規化を経て、最終的に164k登録表面メッシュ（MNI Volume Space）、32k登録表面メッシュ（MNI Volume Space、Native Volume Space）を生成する。*

### 04. GenericfMRIVolume（fMRIボリューム処理）

HCP Minimal Preprocessing Pipelines の4本目のパイプラインは
fMRI Volume pipelineである。
このpipeline は「GenericfMRIVolumeProcessingPipelineBatch.sh」から呼び出される。

このPreprocessingでは、次のような事が行われる。
  * gradient-nonlinearity-induced distortionの補正
  * 体動補正のための時系列データの再調整と関連パラメータの出力
  * fMRI画像の位相エンコード方向の歪みをtopup、FLIRT、BBRを使って補正
  * fMRI画像のT1強調画像への合わせ込み
  * 被験者固有空間から標準空間への合わせ込み

fMRIVolume Pipeline の出力は 主にT1wフォルダの下のResultsフォルダへ出力される。

**目的**: fMRI時系列の前処理（ボリューム空間）

**主要処理**:
- モーション補正（MCFLIRT/FLIRT）
- 磁化率歪み補正（TOPUP使用）
- SBRef画像を用いた精密レジストレーション
- T1w空間およびMNI空間への変換

**データバリエーション**:
- SingleSetFMri: 1st_REST_AP + 1st_REST_PA
- DoubleSetFMri: 2セットのREST
- TripleSetFMri: 3セットのREST

**主要パラメータ**:
- 最終解像度: 2mm
- バイアス補正: SEBASED
- モーション補正: MCFLIRT


![fMRI Volume Pipeline Workflow](img/glasser2013_fig19.png)
*図10: fMRI Volume Pipeline処理フロー (Glasser et al., 2013) - Spin Echo Field Map（LR/RL）、Original Timeseries、SBRef Imageから開始し、Gradient歪み補正、SE EPI Field Map前処理、Motion Correction to SBRef、EPI Image歪み補正を経て、SBRefからT1w BBRレジストレーション、T1w/MNI Reg、One Step Spline Resampling、Intensity正規化・Brain Maskingまでの処理フロー。*

### 05. GenericfMRISurface（fMRI表面処理）  

HCP Minimal Preprocessing Pipelines の５本目のパイプラインは fMRI Surface Pipeline である。
機能的MRIの前処理の２つ目になる。
このPipelineは GenericfMRISurfaceProcessingPipelineBatch.sh から呼び出される。

このPreprocessingでは、次のような事が行われる。
  * fMRI画像の脳表リボン上への投写と標準脳への合わせ込み、
  * fMRI Volume画像のデータを被験者脳表画像に配置する(Volume-Surfaceマッピング)
  * fMRIデータの外れ値の除去、脳表面上での平滑化（FWHM2mm）
  * 皮質下灰白質領域の標準脳への合わせ込み
  * 両半球の「表面時系列」と「皮質下体積時系列」のCIFTI形式データの生成
である

**目的**: fMRIデータの表面投影

**主要処理**:
- ボリュームから表面への投影
- 表面スムージング
- Ribbon-constrained投影



fMRISurface Pipeline の出力は 主に MNINonLinearフォルダのしたのResultsフォルダ下へなされる。

![fMRI Surface Pipeline Workflow](img/glasser2013_fig20.png)
*図11: fMRI Surface Pipeline処理フロー (Glasser et al., 2013) - MNI空間のVolume TimeseriesとParcel Constrained Resampling（Individual to Atlas皮質下Voxelを2mm FWHMで処理）から、Cortical Ribbon-based Volume to Surface Mapping（Native Mesh上でNoisy Voxelを除外）、32k登録メッシュへの表面リサンプリング、2mm FWHM Gaussian表面スムージングを経て、CIFTI Dense Timeseries（91282 Grayordinates）を生成。*

### 06. IcaFix（ICA+FIX）
**目的**: 独立成分分析によるアーチファクト除去

**主要処理**:
- ICA分解による成分抽出
- FIXによる自動ノイズ成分分類
- ノイズ成分の回帰除去

**設定例**:
- 連結名: "Single_PA_AP"
- バンドパス: 0（線形デトレンド）
- FIX閾値: 10
- トレーニングデータ: HCP_Style_Single_Multirun_Dedrift.RData

### 07. PostFix（Post-FIX処理）
**目的**: FIX処理後のクリーンアップ

**主要処理**:
- 清浄化されたfMRIデータの標準化
- 表面・ボリューム統合

### 08. MSMAll（マルチモーダル表面マッチング）
**目的**: 個人間の皮質領域対応の最適化

**主要処理**:
- 皮質形状、機能、接続性を統合した表面レジストレーション
- より精密な群解析のための対応付け

### 09. DeDriftAndResample（ドリフト補正・リサンプリング）
**目的**: 時系列ドリフトの除去と最終リサンプリング

**主要処理**:
- 線形・高次ドリフト補正
- 標準メッシュへの最終リサンプリング
- データの標準化

### 10. RestingStateStats（安静時fMRI統計）
**目的**: 安静時fMRIの統計解析と品質評価

**主要処理**:
- 皮質パーセル化（Glasser 360等）
- 接続行列計算
  * `MNINonLinear/Results/Single_PA_AP/Single_PA_AP_Atlas_stats.txt`
  * `MNINonLinear/Results/Single_PA_AP/Single_PA_AP_Atlas_stats.dscalar.nii`
- 信号品質メトリクス算出

**出力パラメータ**:
- 低解像度メッシュ: 32k
- スムージング: 2mm FWHM  
- 出力識別子: "_hp0_clean"



### 11. Diffusion Preprocessing Pipeline

HCP Minimal Preprocessing Pipelines の11本目のパイプラインは
Diffusion Preprocessing Pipeline である。
このパイプラインは DiffusionPreprocessingBatch.sh から呼び出される。

このPreprocessingでは、次のような事が行われる。
  * 拡散強調信号の標準化
  * 各種歪み補正
  * Diffusion MRIを構造画像へ合わせ込む
  * 空間解像度修正
  * 脳マスク画像の適応 

Diffusion Preprocessing Pipelineの出力は主に T1wフォルダの下のDiffusionフォルダになされる。

#### 備考

MP-PCA（Marchenko-Pastur主成分解析）をベースとしたノイズ除去、
ギブスリンギングアーティファクト除去、ANTsベースバイアス補正を加えた改良パイプラインを整備中。

なお、Diffusion preprocessingのあとの適切な解析は、未だ公開されておらず、開発中の状態である。
FSLのbedpostxやMRtrix3等を使って処理することで白質線維関連の特徴量の推定等を行えるが、
明らかなコンセンサスは得られていません。
ここは独自の拡張に挑戦する価値がある部分と思われる。

Diffusion preprocessingのあとは適切な解析が未だ公開されておらずずっと開発中である。（NODDI, DTI、トラクトグラフィーなど）。NODDIについてはFukutomi et al., 2018が臨床研究への応用にもっとも近い技術だと考えている。

![Diffusion Preprocessing Pipeline Workflow](img/glasser2013_fig22.png)
*図12: Diffusion Preprocessing Pipeline処理フロー (Glasser et al., 2013) - Original DiffusionTimeseries（LR）とOriginal Diffusion Timeseries（RL）から開始し、b0 Intensity正規化、Subject Motion Correction（eddy）、Gradient非線形性補正、b0 to T1w BBR レジストレーション、1.25mm Native Structural Spaceへのリサンプリング、T1w/Maskを経て、最終的な拡散データを生成。*


以上である。
