# 小池研勉強会資料：HCP Pipelinesを走らせて、特徴量を取り出す

2025.09.12

--------

## この資料の内容

  * HCPpipelinesのスクリプトの組み方（sMRI部分のみ）
  * HCPpipelines出力ファイルから基本的な特徴量を取り出す方法
  * 応用的な特徴量を取り出す方法（時間が許せば）

--------

## スクリプトを組み方

HCPpipelinesにおいてsMRI処理は `PreFreeSurferPipeline`、`FreeSurferPipeline`、`PostFreeSurferPipeline` のステップで行われる。この３ステップはfMRI、dMRI処理の基盤ともなりHCPpipelinesの中心的処理になる。
これらステップを走らせるためのスクリプトの組み方を紹介する。

(based on release 4.5.0 )

### 01_PreFS

`HCPpipelines/Examples/Scripts/PreFreeSurferPipelineBatch.sh` を自施設用に改造して使う。
コアな部分は以下の部分。

```bash
${HCPPIPEDIR}/PreFreeSurfer/PreFreeSurferPipeline.sh \
  --path="$StudyFolder" \
  --subject="$Subject" \
  --t1="$T1wInputImages" \
  --t2="$T2wInputImages" \
  --t1template="$T1wTemplate" \
  --t1templatebrain="$T1wTemplateBrain" \
  --t1template2mm="$T1wTemplate2mm" \
  --t2template="$T2wTemplate" \
  --t2templatebrain="$T2wTemplateBrain" \
  --t2template2mm="$T2wTemplate2mm" \
  --templatemask="$TemplateMask" \
  --template2mmmask="$Template2mmMask" \
  --brainsize="$BrainSize" \
  --fnirtconfig="$FNIRTConfig" \
  --fmapmag="$MagnitudeInputName" \
  --fmapphase="$PhaseInputName" \
  --fmapgeneralelectric="$GEB0InputName" \
  --echodiff="$TE" \
  --SEPhaseNeg="$SpinEchoPhaseEncodeNegative" \
  --SEPhasePos="$SpinEchoPhaseEncodePositive" \
  --seechospacing="$SEEchoSpacing" \
  --seunwarpdir="$SEUnwarpDir" \
  --t1samplespacing="$T1wSampleSpacing" \
  --t2samplespacing="$T2wSampleSpacing" \
  --unwarpdir="$UnwarpDir" \
  --gdcoeffs="$GradientDistortionCoeffs" \
  --avgrdcmethod="$AvgrdcSTRING" \
  --topupconfig="$TopupConfig" 
```
  
このスクリプトに渡すべき変数は `HCPpipelines/Examples/Scripts/` 下を読み解くとなんとなく見えてくるが、整理すると以下の通り。

* `--path`: 作業フォルダ
* `--subject`: 被験者ID
* `--t1`: T1WI NIfTIファイルへのパス
* `--t2`: T2WI NIfTIファイルへのパス。なければ "NONE"。
* `--t1template`: Hires brain extracted MNI template. `MNI152_T1_0.7mm.nii.gz`.
* `--t1templatebrain`: Hires brain extracted MNI template. `MNI152_T1_0.7mm_brain.nii.gz`.
* `--t1template2mm`: Lowres T1w MNI template. `MNI152_T1_2mm.nii.gz`. 
* `--t2template`: Hires T2w MNI Template. `MNI152_T2_0.7mm.nii.gz`.
* `--t2templatebrain`: Hires T2w brain extracted MNI Template. `MNI152_T2_0.7mm_brain.nii.gz`.
* `--t2template2mm`: Lowres T2w MNI Template. `MNI152_T2_2mm.nii.gz`.
* `--templatemask`: Hires MNI brain mask template. `MNI152_T1_0.7mm_brain_mask.nii.gz`.
* `--template2mmmask`: Lowres MNI brain mask template. `MNI152_T1_2mm_brain_mask_dil.nii.gz`.
* `--brainsize`: BrainSize in mm, 150 for humans. 
* `--fnirtconfig`: FNIRT用設定ファイルへのパス。既定のもの（`${HCPPIPEDIR_Config}/T1_2_MNI152_2mm.cnf`）で良い。
* `--fmapmag`: GEFMap を使う場合に設定。SEFMapを使う場合は "NONE" で良い。
* `--fmapphase`: GEFMap を使う場合に設定。SEFMapを使う場合は "NONE" で良い。
* `--fmapgeneralelectric`: GEFMap を使う場合に設定。SEFMapを使う場合は "NONE" で良い。
* `--echodiff`: GEFMap を使う場合に設定。SEFMapを使う場合は "NONE" で良い。
* `--SEPhaseNeg`: LR または AP SEFMap NIfTI画像画像のパス。SEFMがなければ、"NONE"。
* `--SEPhasePos`: RL または PA SEFMap NIfTI画像画像のパス。SEFMがなければ、"NONE"。
* `--seechospacing`: 
Spin Echo Field Maps の Effective Echo Spacing を秒単位で指定する。
計算式は「SEEchoSpacing = 1 / (BWPPPE * ReconMatrixPE)」。
EchoSpacingとは、隣接するk-spaceラインの読み取り間隔。
BWPPPE (BandwidthPerPixelPhaseEncode) は、位相エンコード方向における 1 ピクセルあたりの受信帯域幅（Hz）。
ReconMatrixPE は再構成画像の位相エンコード方向のマトリクスサイズ。
使用するスキャナやシーケンスごとに異なるので、実データの DICOM header から計算して埋め込む必要がある。
dcm2niixで NIfTI変換した場合には、同時に出力されるjsonファイルに BandwidthPerPixelPhaseEncode、ReconMatrixPE、EffectiveEchoSpacing が書かれている。
ちなみに HCP-YAデータでは "0.00058"。SEFMがなければ、"NONE"。
<!-- https://chatgpt.com/c/68bfac15-e350-8330-953c-bc4e19cfbf05 -->

* `--seunwarpdir`: 
SEFMの位相エンコード方向 を指定。位相エンコード方向は EPI 系の撮像で最も歪みやすい方向を表す。
この方向を指定することで、topup による歪み補正が正しく働き、T1w/T2w 画像の幾何学的歪みが修正される。
左右方向（LR/RL）であれば、`x` or `j`。
前後方向（AP/PA）であれば、`y` or `i`。
`+x` だとか  `j-` だとか、符号は付けないこと。
SEFMがなければ、"NONE"。
<!--https://chatgpt.com/c/68bfb1d8-a284-8330-9256-af3de12bf963-->
* `--unwarpdir`: 
構造画像（T1w, T2w）のリードアウト（信号が読み出される軸）方向。
`--seunwarpdir`との混同注意！。
ここでは 符号の有無が重要となる（例: z と z- は別の意味）。
Siemens の 3D T1w/T2w：MPRAGE/SPACE なら "z" で良い。
それ以外の場合は、dcm2niix出力jsonの MRAcquisitionType, InPlanePhaseEncodingDirectionDICOM, ImageOrientationPatientDICOMから特定する。
「AvgrdcSTRING="NONE"」と指定したときは "NONE" で良い。
<!--https://chatgpt.com/c/68bfb791-d600-8333-bc82-686f2df16927-->
* `--t1samplespacing`: 
T1w の 位相エンコード方向で 1 ピクセルを読むのにかかる時間（DwellTime）。
DICOM field (0019,1018) の 値、もしくは dcm2niix出力JSON にある DwellTime を秒で入れる。
DwellTime が JSON に無い場合は、1 / (PixelBandwidth * ReconMatrixPE)（≈ NumberOfPhaseEncodingSteps）で計算できる（単位は秒）。
* `--t2samplespacing`:
T2w の 位相エンコード方向で 1 ピクセルを読むのにかかる時間。
DICOM field (0019,1018) の 値、もしくは dcm2niix出力JSON にある DwellTime を秒で入れる。
DwellTime が JSON に無い場合は、1 / (PixelBandwidth * ReconMatrixPE)（≈ NumberOfPhaseEncodingSteps）で計算できる（単位は秒）。T2WIがないならば "NONE"。
<!--https://chatgpt.com/c/68bfbdba-2548-8323-9a4a-1402b45bbf8d-->
* `--gdcoeffs`: ベンダーから入手。手に入らなければ "NONE" とすれば GDCはスキップ。
* `--avgrdcmethod`: Readout Distortion Correctionの方法を指定する。"SiemensFieldMap", "FIELDMAP", "GeneralElectricFieldMap", "TOPUP", or "NONE" から選ぶ。FSLの topupt を使いたい場合は "TOPUP" を指定。Siemens の Gradient Echo Field Map を使いたい場合は "SiemensFieldMap" or "FIELDMAP" を指定。GE の Gradient Echo Field Map を使いたい場合は "GeneralElectricFieldMap" を指定。 Readout Distortion Correction を省く場合は "NONE"。GEFMを使った補正をする場合は、`$PhaseInputName`、`$GEB0InputName`、$TE`といった変数に "NONE" 以外を指定する必要がある。
<!--https://chatgpt.com/c/68bfc2e4-cf24-832e-9931-2eeddd7beab1-->
* `--topupconfig`: "$HCPPIPEDIR/global/config/b02b0.cnf"。これは FSL が提供する既定の topup 設定ファイルで、施設ごとに作り直す必要は基本的にはない。
<!--https://chatgpt.com/c/68bfb490-0378-8322-bfae-27c847d58bfa-->



### 02_FS

`HCPpipelines/Examples/Scripts/FreeSurferPipelineBatch.sh` を自施設用に改造して使う。
コアな部分は以下の部分。

```bash
${HCPPIPEDIR}/FreeSurfer/FreeSurferPipeline.sh \
  --subject="$Subject" \
  --subjectDIR="$SubjectDIR" \
  --t1="$T1wImage" \
  --t1brain="$T1wImageBrain" \
  --t2="$T2wImage" 
```

このスクリプトに渡すべき変数は整理すると以下の通り。

* `--subject`: 被験者ID
* `--subjectDIR`: "\${StudyFolder}/\${Subject}/T1w"。FreeSurferをデフォルト設定で単体で走らせたときに `SUBJECT_DIR` と解釈される場所に相当する。
* `--t1`: "T1w/T1w_acpc_dc_restore.nii.gz"
* `--t1brain`: "T1w/T1w_acpc_dc_restore_brain.nii.gz"
* `--t2`: "T1w/T2w_acpc_dc_restore.nii.gz"

acpc = 剛体変換によりACPC空間に位置合わせされた画像; 
brain = 脳だけが抽出された画像; 
estore = バイアスフィールド画像（BiasField.nii.gz）を使って信号値補正された画像; 
dc = Readout Distortion Corrected.


### 03_PostFS

`HCPpipelines/Examples/Scripts/PostFreeSurferPipelineBatch.sh` を自施設用に改造して使う。
コアな部分は以下の部分。

```bash
${HCPPIPEDIR}/PostFreeSurfer/PostFreeSurferPipeline.sh \
      --study-folder="${StudyFolder}"  \
      --subject="${Subject}" \
      --surfatlasdir="${HCPPIPEDIR_Templates}/standard_mesh_atlases"   \
      --grayordinatesdir="${HCPPIPEDIR_Templates}/91282_Greyordinates"   \
      --grayordinatesres=2   \
      --hiresmesh=164   \
      --lowresmesh=32   \
      --subcortgraylabels="${HCPPIPEDIR_Config}/FreeSurferSubcorticalLabelTableLut.txt"   \
      --freesurferlabels="${HCPPIPEDIR_Config}/FreeSurferAllLut.txt"   \
      --use-ind-mean="$UseIndMean"　\
      --refmyelinmaps="${HCPPIPEDIR_Templates}/standard_mesh_atlases/Conte69.MyelinMap_BC.164k_fs_LR.dscalar.nii" \
      --regname="$RegName"
```

* `--study-folder`: 複数の被験者サブディレクトリを含むベースディレクトリを指定。
* `--subject`: 被験者ID もしくは セッションID。
* `--surfatlasdir`: 各種標準脳メッシュを格納しているフォルダ
* `--grayordinatesdir`: Grayordinates 空間（例えば 91282_Greyordinates）のテンプレートフォルダ。
* `--grayordinatesres`: Grayordinates の解像度（単位：mm）。デフォルトは 2（2 mm）。
* `--hiresmesh=164`: T1w 解像度用の高解像度メッシュサイズ。通常164で良い。
* `--lowresmesh=32`: fMRI 用に用いられる低解像度メッシュサイズ。通常32で良い。
* `--subcortgraylabels`: FreeSurfer のサブコルティカルラベルLUTファイルの位置（HCPpipelines/global/config/FreeSurferSubcorticalLabelTableLut.txt）
* `--freesurferlabels`: FreeSurfer ラベル LUT ファイル位置（HCPpipelines/global/config/FreeSurferAllLut.txt）
* `--use-ind-mean`:  
脳部位ごとに生じる系統的な強度の歪みを補正するためのオプション。"use individual mean" を意味する。
YES → 各被験者自身の myelin map の平均値を用いる。
NO → --refmyelinmaps で指定したグループ平均を基準にする。
* `--refmyelinmaps`: 
脳部位ごとに生じる系統的な強度の歪みを補正するためのグループ平均ミエリンマップを指定するオプション。通常はHCP標準集団で作られた "standard_mesh_atlases/Conte69.MyelinMap_BC.164k_fs_LR.dscalar.nii" を指定する。独自集団の平均マップを基準に補正したい場合は独自ファイルを渡しても良い。
* `--regname`: 登録方法の名前。"MSMSulc" や "FS" を指定する。ただしここで指定した文字列によってレジストレーション方法が切り替わる訳ではなく、出力ファイル名に含まれる文字列が変化するのみ。
* `--processing-mode`: "HCPStyleData" または "LegacyStyleData" を指定する。"HCPStyleData" とした場合は、HCP準拠（高分解能 T1w+T2w、fMRI/dMRI は適切な歪み補正用データ等）を前提に処理する。準拠していない入力が来るとエラー／停止する。"LegacyStyleData" を指定すると HCP準拠を満たさない（例：T2wが無い、フィールドマップが無い等）データでも、可能な範囲で処理を継続できる。

---------

## 出力ファイルを把握する

### 画像ファイル形式：DICOM、NIfTI、GIfTI、CIfTI

ここのところは2024.09.06 でも説明しているので詳しくはそちらを参照のこと。
ここでは後の説明で重要となる以下のみ復習しておく。

  * pscalar.nii: 脳機能や構造に関するデータを格納するためのCIFTI形式の一種で、特に特定の脳領域（パーセル、parcel）におけるスカラー値（例えば、統計的なパラメータや時系列の平均値など）を格納する。
  * dscalar.nii: 脳機能や構造に関するデータを格納するためのCIFTI形式の一種で、脳全体の各頂点（大脳皮質の表面メッシュ上の点）や各ボクセル（サブ皮質領域を含む）ごとにスカラー値を格納する。
  * func.gii: メトリックファイル。頂点に関連付けられた連続値を格納するGIfTIファイルの一種。
  * shape.gii: ジオメトリファイル。頂点座標と三角形配列の情報を格納するGIfTIファイルの一種。

### HCPpiepelines処理後のファイル・フォルダ

#### T1w/ と MNINonLinear/

HCPpiplinesを走らせる以下のような構造でフォルダが生成される。
被験者名を起点に第一階層にMNINonLinearというフォルダとT1wというフォルダが作られる。

```
100307/
 +-- MNINonLinear/
 |    +-- ./     ④
 |    +-- Native/    ③
 |    +-- fsaverage_LR32k/    ⑤
 |    +-- Results/
 |    :
 +-- T1w/
 |    +-- ./    ①
 |    +-- 100307/
 |    +-- Native/    ②
 |    +-- fsaverage_LR32k/    ⑥
 |    +-- Results/
 |    +-- Diffusion/
 :    :
```

* `MNINonLinear/`​
  * MNI152テンプレートに合わせこまれてたデータが格納されている。
  * 異なる被験者であっても大まかな脳のサイズは同一になる。​
  * 皮質下構造の体積やBOLD信号などの被験者固有の皮質形態とは直接関係ない特徴量を計測したい場合はこちらのフォルダ下にあるものを使う。
    * MNINonLinear/下のファイルでは、皮質下データが非線形ボリューム登録によって皮質データよりも正確に位置合わせされているため。
  * 解剖学的位置情報が標準化されているため、voxelレベル、vertexレベルで異なる被験者間・研究間で比較する差異に都合が良い。​

* `T1w/`
  * 被験者の元々のT1強調画像に合わせこまれたデータが格納されている。
  * 被験者固有の脳の大きさが保たれているので、被験者が異なると大まかな脳のサイズも異なる。
  * 皮質の形態由来情報（皮質厚、皮質体積、皮質面積等）、トラクトグラフィーなど被験者固有の皮質形態を保った上で特徴量を計測したい場合はこちらのフォルダ下にあるものを使う。
  * 解剖学的位置情報が標準化されていないため異なる被験者間・研究間で比較するためには、被験者脳をレジストレーションするための共通テンプレートやアトラスを用いるなどの工夫が必要である。​



#### T1w/直下、MNINonLinear/直下 にあるファイルについて

* `T1w/直下`(①): 
  * NIfTI形式の脳画像ファイルが置かれる。
  * このNIfTIには被験者の脳の大きさがそのまま保たれたボリュームデータが入っている。

* `MNINonLinear/直下`(④): 
  * NIfTI、GIfTI、CIfTI形式の脳画像ファイルが置かれる。​
  * ここのNIfTIは①にあるNIfTIに格納されたボリュームデータが、MNI152テンプレートに合わせ込まれたボリュームデータである。​
  * ここのGIfTIには③のサーフェスデータデータをリサンプリングしてConte69サーフェステンプレート（頂点数は高解像度（16万4千頂点））に合わせ込んだものが格納されている。​
  * ここのCIfTIには、 NIfTIからの皮質下体積情報とGIfTIからの皮質情報が格納されている。​

* 備考: 
acpc = 剛体変換によりACPC空間に位置合わせされた画像を意味; 
brain = 脳だけが抽出された画像を意味; 
restore = バイアスフィールド画像（BiasField.nii.gz）を使って信号値補正された画像を意味; 
dc = Readout Distortion Corrected を意味; 
gdc: Gradient Distortion Corrected (Gradient Nonlinearity Corrected) を意味; 
BiasField.nii.gz = T1wおよびT2w画像より作成したバイアスフィールドの画像; 
brainmask_fs.nii.gz = FreeSurferにより推定した脳部分が１となっているバイナリー画像; 
ribbon.nii.gz = 皮質部分のROI画像; 
wmparc.nii.gz = FreeSurferにより推定した脳分画の画像; 
T1wDividedByT2w.nii.gz = T1WIをT2WIで除算した画像（ミエリンマップの元画像）

#### */Native/ にあるファイルについて
* `T1w/Native/`(②): 
  * Native Surface Mesh in Native Volume Spaceが置かれる。
    * HCPpipelinesの設計の中では FreeSurfer処理で得られたMeshを Native Surface Mesh と呼んでいる。
  * すなわち、①にある NIfTIに格納されたボリュームデータがFreeSurfer処理されて、そこから得られたサーフェスデータがGIfTIファイルとして置かれる。​
* `MNINonLinear/Native/`(③ ): 
  * Native Surface Mesh in MNI Volume Spaceが置かれる。
  * すなわち、②にあるのsurfaceが位置・サイズ変換されたものが置かれる。​
​

#### */fsaverage_LR32k/ にあるファイルについて

* `MNINonLinear/fsaverage_LR32k/`(⑤): 
  * ④にある高解像度（約16万4千頂点）で表されたサーフェスデータをダウンサンプリングし低解像度（約3万2千頂点）で表したサーフェスデータが置かれる。
  * これはfMRI解析に向いたデータとなる。​

* `T1w/fsaverage_LR32k`(⑥): 
  * ②下にあるサーフェスデータが低解像度（約3万2千頂点）にresamplingされたものが置かれる。​

#### その他の場所にあるファイルについて

* `MNINonLinear/Results/` には、fMRI前処理結果が格納されている。fMRIのSurface-Based解析用。
* `T1w/100307/` にはFreeSurfer処理結果が格納されている。
* `T1w/Results/` には、fMRI前処理結果（T1強調画像に合わせこまれた前処理後のfMRIデータ）が格納されている。QC用と思われる。またT1wに合わせこまれた状態でfMRI解析を行いたい場合に使えるかもしれない。
* `T1w/Diffusion/` には Diffusion Preprocessing Pipeline処理結果が格納されている。


--------

## 脳特徴量を取り出す方法（基本編）

2025.09現在、小池研で使っているスクリプト（
Aggregate_ssMRI_Features_on_MMP1/ForRelease/codes/Aggregate_sMRI_NIDPs_on_MMP1_with_MSMSulc.sh）から
各種の基本的な特徴量の抽出方法の要点を紹介する。

以下に示すコマンドによって *.pscalar.nii ファイルが得られるので、これを *.csv ファイル等に変換して解析に使うことになる
（*.dscalar.nii ⇒ *.pscalar.nii ⇒ *.csv）。

**備考**
*  `${PARCEL_FILE}` は `Glasser_et_al_2016_HCP_MMP1.0_v6_RVVG/Q1-Q6_RelatedValidation210/MNINonLinear/fsaverage_LR32k/Q1-Q6_RelatedValidation210.CorticalAreas_dil_Final_Final_Areas_Group_Colors.32k_fs_LR.dlabel.nii` であり、その入手先は [https://balsa.wustl.edu/DLabel/3VLx](https://balsa.wustl.edu/DLabel/3VLx) である。
* MSMAllレジストレーション後のファイルを使う場合は、入力ファイルをMSMAll処理後のものに置き換える必要がある。（See Aggregate_ssMRI_Features_on_MMP1/ForRelease/codes/Aggregate_sMRI_NIDPs_on_MMP1_with_MSMSulc.sh）


### **Cortical Thickness (CT)**

```bash
CIFTI_DSCALAR_METRIC=MNINonLinear/fsaverage_LR32k/100307.thickness.32k_fs_LR.dscalar.nii

WEIGHT_FILE_FOR_LH=T1w/fsaverage_LR32k/100307.L.midthickness.32k_fs_LR.surf.gii
WEIGHT_FILE_FOR_RH=T1w/fsaverage_LR32k/100307.R.midthickness.32k_fs_LR.surf.gii

CIFTI_PSCALAR_OUT=NIDPs/100307_CT_MSMSulc.pscalar.nii

wb_command -cifti-parcellate \
    ${CIFTI_DSCALAR_METRIC} ${PARCEL_FILE} COLUMN \
    ${CIFTI_PSCALAR_OUT} \
    -spatial-weights \
    -left-area-surf ${WEIGHT_FILE_FOR_LH} -right-area-surf ${WEIGHT_FILE_FOR_RH} \
    -method MEAN

# Points:  
#   * COLUMNは、パーセル化対象情報が CIFTI_DSCALAR_METRIC のどの軸にあるかを示すための指定。
#   * "-spatial-weights" のオプションを指定してT1w/fsaverage_LR32k以下のmidthickness関連のファイルを指定し
#     "-method MEAN" を指定して、重み付け平均（weighted mean）を算出。
```

### **Surface Area (SA)**

```bash
CIFTI_DSCALAR_METRIC=T1w/fsaverage_LR32k/100307.midthickness_va.32k_fs_LR.dscalar.nii

CIFTI_PSCALAR_OUT=NIDPs/100307_SA_MSMSulc.pscalar.nii

wb_command -cifti-parcellate \
    ${CIFTI_DSCALAR_METRIC} ${PARCEL_FILE} COLUMN \
    ${CIFTI_PSCALAR_OUT} \
   -method SUM

# Points: 
#   * midthickness_va は  個人の「絶対的」局所表面積（mm²）。
#   * "-method SUM" を指定して、総計値を算出。
#   * 重み付け不要なので、"-spatial-weights" は指定しない。
#   * ConnectomeDBから入手したPreprocデータには、
#    「T1w/fsaverage_LR32k/100307.midthickness_va.32k_fs_LR.dscalar.nii」が無いので注意のこと。
```

### **Normalized Surface Area (NSA)**

```bash
CIFTI_DSCALAR_METRIC=T1w/fsaverage_LR32k/100307.midthickness_va_norm.32k_fs_LR.dscalar.nii

CIFTI_PSCALAR_OUT=NIDPs/100307_NSA_MSMSulc.pscalar.nii

wb_command -cifti-parcellate \
    ${CIFTI_DSCALAR_METRIC} ${PARCEL_FILE} COLUMN \
    ${CIFTI_PSCALAR_OUT} \
    -method SUM

# Points: 
#   * midthickness_va_norm は 全皮質で正規化された「相対的」局所表面積（単位なし）。
#   * ""-method SUM" を指定して、総計値を算出。
#   * 重み付け不要なので、"-spatial-weights" は指定しない。
#   * ConnectomeDBから入手したPreprocデータには、
#    「T1w/fsaverage_LR32k/100307.midthickness_va_norm.32k_fs_LR.dscalar.nii」が無いので注意のこと。
```


### **Cortical Volume (CV**)

```bash

# Create Metric File (MEASURE PER-VERTEX VOLUME BETWEEN SURFACES)

# Calc vol between two surfaces for Left
wb_command -surface-wedge-volume \
    T1w/fsaverage_LR32k/100307.L.white.32k_fs_LR.surf.gii \
    T1w/fsaverage_LR32k/100307.L.pial.32k_fs_LR.surf.gii \
    T1w/fsaverage_LR32k/L.cortical_volume.func.gii

# Calc vol between two surfaces for Right
wb_command -surface-wedge-volume \
    T1w/fsaverage_LR32k/100307.R.white.32k_fs_LR.surf.gii \
    T1w/fsaverage_LR32k/100307.R.pial.32k_fs_LR.surf.gii \
    T1w/fsaverage_LR32k/R.cortical_volume.func.gii

# CREATE CIFTI WITH MATCHING DENSE MAP
template_cifti_dscalar=MNINonLinear/fsaverage_LR32k/100307.thickness.32k_fs_LR.dscalar.nii
gifti_metric_L=T1w/fsaverage_LR32k/L.cortical_volume.func.gii
gifti_metric_R=T1w/fsaverage_LR32k/R.cortical_volume.func.gii
CIFTI_DSCALAR_METRIC=MNINonLinear/fsaverage_LR32k/100307.CV_MSMSulc.32k_fs_LR.dscalar.nii

wb_command -cifti-create-dense-from-template \
    ${template_cifti_dscalar} ${CIFTI_DSCALAR_METRIC} \
    -metric CORTEX_LEFT ${gifti_metric_L} \
    -metric CORTEX_RIGHT ${gifti_metric_R}

CIFTI_PSCALAR_OUT=NIDPs/100307_CV_MSMSulc.pscalar.nii

wb_command -cifti-parcellate \
    ${CIFTI_DSCALAR_METRIC} ${PARCEL_FILE} COLUMN \
    ${CIFTI_PSCALAR_OUT} \
    -method SUM

# Points: 
#   * 白質と軟膜の間の間から体積を計測する
#   * このやり方は HCP Users GoogleGroupsに説明がある。https://groups.google.com/a/humanconnectome.org/g/hcp-users/c/wUQtOO340e8/m/l9LIvI4-EwAJ
```

### **Myelin Map (MM)**

```bash
CIFTI_DSCALAR_METRIC=MNINonLinear/fsaverage_LR32k/100307.MyelinMap.32k_fs_LR.dscalar.nii

WEIGHT_FILE_FOR_LH=T1w/fsaverage_LR32k/100307.L.midthickness.32k_fs_LR.surf.gii
WEIGHT_FILE_FOR_RH=T1w/fsaverage_LR32k/100307.R.midthickness.32k_fs_LR.surf.gii

CIFTI_PSCALAR_OUT=NIDPs/100307_MM_MSMSulc.pscalar.nii

wb_command -cifti-parcellate \
    ${CIFTI_DSCALAR_METRIC} ${PARCEL_FILE} COLUMN \
    ${CIFTI_PSCALAR_OUT} \
    -spatial-weights \
    -left-area-surf ${WEIGHT_FILE_FOR_LH} -right-area-surf ${WEIGHT_FILE_FOR_RH} \
    -method MEAN

# Points: 
#   * "-spatial-weights" のオプションを指定してT1w/fsaverage_LR32k以下のmidthickness関連のファイルを指定し
#     "-method MEAN" を指定して、重み付け平均（weighted mean）を算出。 
```

<!--Sulcal Depth ←これはどうやって？-->

<!--Curvature ←これはどうやって？-->

-------

## 脳特徴量を取り出す方法（応用編）

DTI特徴量、NODDI特徴量、LGI特徴量など応用的な特徴量をHCPpipelinesのSBAに基づいて取り扱う時、`wb_command -metric-resample` を使う必要がある。​

このコマンドが何をしているかを理解するには、HCPpipelines処理後のメッシュファイル達とそれらの相互関係の理解が必要である。

メッシュファイルへの理解が深まると、応用的な特徴量の算出スクリプトが組めみやすくなるだけでなく、
メッシュファイルへの理解が深まると、HCPpipelinesが採用しているMSMが、VBAレジストレーションやFreeSurferレジストレーションよりも細かな個人差を保持するように働いているという実感が持てる。

### HCPpipelines処理で生成されるメッシュファイル達

  * Overview
  * サイズ感違い: midthickness of MNINonLinear and T1w​
  * 頂点の違い: 32k vs 164k vs Native ​
  * Population Average Sphere: fs_LR球 と fsevrage球 の違い​
  * Original球 と FreeSurfer球 と MSMSulc球 と MSMAll球 ​
  * fs_LR deformed to fsaverage sphere とは？Standard fs_LR sphereとの比較 ​
  * 個人のfsLR-164kと標準fsLR-164k ​
  * 個人のfsLR-32kと標準fsLR-32k ​

See [2025.09.12_Metric-Resampling-3.pdf](./2025.09.12_Metric-Resampling-3.pdf)

  
### 脳表にある特徴量をリサンプリングするとは

  * 模式図
  * CTを例にした Metric Resampling
  * LGIを例にした Metric Resampling

See [2025.09.12_Metric-Resampling-3.pdf](./2025.09.12_Metric-Resampling-3.pdf)


### LGI on MMP1 

https://github.com/iueda123/Aggregate_LGI_on_MMP1 （2025.09.12 時点で非公開リポジトリ）

大まかな流れ

* 1. `mris_convert` による shape.giiの用意
```bash
# 処理コア
mris_convert \
    -c ${subject_root_on_prc}/T1w/${sbj_id}/surf/lh.pial_lgi \
    ${subject_root_on_prc}/T1w/${sbj_id}/surf/lh.pial \
    ${subject_root_on_prc}/T1w/Native/${sbj_id}.L.pial_lgi.native.shape.gii
```

* 2. `wb_command -metric-resample` による shape.gii ⇒ dscalar.nii
* 3. `wb_command -cifti-parcellate` による dscalr.nii ⇒ pscalar.nii
* 4. `wb_command -cifti-convert` による pscalr.nii ⇒ csv


### DTI、NODDI on MMP1

https://github.com/iueda123/Aggregate_dMRI_Features_on_MMP1（2025.09.12 時点で非公開リポジトリ）

https://github.com/RIKEN-BCIL/NoddiSurfaceMapping と 小池件渋川先生情報  と 昭和医科大学発達障害医療研究所
板橋貴史 先生  情報を参考に組み立てた。

**大まかな流れ**  
* 1. DTIの FA、MD、NODDIのFICVF、ODI、Kappa、データ信号対雑音比などの指標を計算（`dtifit`、`amico` 使用）
* 2. Volume → Surface。`wb_command -volume-to-surface-mapping` で ボリューム空間の拡散指標を大脳皮質表面にマッピング ⇒ native.func.gii作成
```bash
# 処理コア
wb_command -volume-to-surface-mapping ${volume_data} \
    ${midthickness_surface} \
    ${output_func_gii} \
    -myelin-style \
    ${ribbon_mask} \
    ${thickness_shape_gii} \
    ${sigma_value}
```
* 3. Resample: wb_command -metric-resampleでnative空間からfsLR空間（32k）に再サンプリング →
func.gii作成
* 4. CIFTI Dense作成: `wb_command -cifti-create-dense-scalar` で左右半球表面+皮質下ボリュームを統合 → dscalar.nii作成
* 5. Parcellation: wb_command -cifti-parcellateでdscalar.niiを脳区画（MMP1等）で平均化 →
pscalar.nii作成
* 6. CSV変換: wb_command -cifti-convertでpscalar.niiを数値テーブル形式に変換 → CSV作成

<!--
### fMRIの加工例

functional connectivity を取り出せる

/media/iu/STORAGE/__StudyData__/Dtbs_ConnectomeDB/HCP-YA/20250903/100307/MNINonLinear/Results/rfMRI_REST1_LR/rfMRI_REST1_LR_Atlas_MSMAll_hp2000_clean.dtseries.nii

https://chat.google.com/dm/5dhE88AAAAE/SBV-HUKcgqw/SBV-HUKcgqw?cls=10

https://chatgpt.com/c/68b80da7-81e8-8333-ad61-b0b3b3d5cf2d

http://isue.dix.asia:8890/dokuwiki/das/doku.php?id=hcppipelines:the_hcp_pipeline-processed_files#t1w

-->

<!--
### Vertex-wise analysis

SBA: CIfTIから頂点ごとの情報を取り出して解析する

cf. SBA CIfTIからMMP1アトラスに從って

VBA 領域ごとの情報を取り出して解析する

-->

-----------

## References 
  * [HCPpipelines@GitHub](https://github.com/Washington-University/HCPpipelines/tree/master)
    * ChatGPT等への尋ね方のコツ：コードを表示させ気になる行をアクティブにし "Copy permalink" を手に入れ尋ねると良い。
  * [HCP-Users@GoogleGroups](https://groups.google.com/a/humanconnectome.org/g/hcp-users)
  * [HCP S1200 Release Reference Manual](https://www.humanconnectome.org/storage/app/media/documentation/s1200/HCP_S1200_Release_Reference_Manual.pdf)
  * [Glasser et al., Neuroimage, 2013](https://pmc.ncbi.nlm.nih.gov/articles/PMC3720813/)
  * 国際脳チュートリアル関連資料/20200927_BMB_Protocols_and_Preproc_Hayashi_Manual.pdf 林拓也先生作成資料
  * [wb_command Documentation](https://www.humanconnectome.org/software/workbench-command)
  
