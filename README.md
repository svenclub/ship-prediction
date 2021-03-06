# **ship-prediction**  
船舶動向予測技術評価のためのプログラム  
**data_sort**:AISデータの分割・ソート  
**Liner**:船舶のオブジェクト化・線形予測時の平均誤差距離・分散計算・座標変換  
**Kalman**:カルマンフィルタを用いた予測時の平均誤差距離・分散計算  
**steepest descent method**:最急降下法利用時のプログラム  
**Jkalman**:カルマンフィルタのライブラリ  
**jama**:行列計算のためのライブラリ

===============
## data_sort  
船舶動向予測技術評価のための一ヶ月分の実データを用意する。  

===============  
### DevidedFiles.java  
デコードされたAIS type1 メッセージをDevideFiles.javaにて一ヶ月毎のcsvファイルに分割する。  
csv形式のAISデータを一行毎に読み込みAISデータのタイムスタンプで１２ヵ月分に分割する。   最終行まで読み込むとプログラムは終了する。  
出力には"年月_メッセージタイプ"をファイル名とするcsvファイルを得る。  
### mmsi_sort.java  
DevidedFiles.javaによって分割されたcsvファイルをさらにMMSI番号毎にソートし新たなcsvファイルに保存する。  
DecidedFiles.javaによって分割されたcsvファイルを１行毎に読み込み、MMSI番号とそれに付随する船舶データを  
配列に保持する。最終行まで読み込むとプログラムは終了する。  
配列に保持されたMMSI番号のcsvファイルが存在すれば、配列に保存された船舶データを当該csvファイルに追加書き込みを  
行う。  
配列に保持されたMMSI番号のcsvファイルが存在しない場合はそのMMSI番号をファイル名とするcsvファイルを新たに作成し  
配列に保存された船舶データを当該csvファイルに書き込みを行う。

===============
## Liner  
座標変換  
船舶のオブジェクト化  
線形予測を行う際の平均誤差距離・分散計算    

===============
### ENU.java  
測地座標系で表現されている船舶GPS位置情報をECEF座標系及びENU座標系に相互変換する。
[理解するためのGPS測位計算プログラム入門](http://www.enri.go.jp/~fks442/K_MUSEN/1st/1st021118.pdf)  
を参考にJavaで各メソッドを作成した。  
#### ChangToECEF(double, double)
測地座標で表される船舶位置をECEF座標に変換する。
2種類のdouble型引数（緯度・経度）をとり、戻り値にdouble型配列(X,Y,Z)を得る。  
#### ChangeToENU(double[][], double[][],double, double)
ECEF座標で表される2点を、一方を原点とするENU座標に変換する。  
double型配列2種類(原点とするECEF座標ともう一方のECEF座標)及びdouble型引数2種類（原点の緯度・経度）を引数にとり  
3行1列のMatrix型(e,n,u)の戻り値を得る。  
""""
#### BacktoECEF(matrix, double[][], double, double)
ENU座標で表される位置をECEF座標に変換する。  
ENU座標が保持される3行1列のMatrix型、ENU座標の原点となるECEF座標が保持されるdouble型配列(X,Y,Z),  
ENU座標の原点となる測地座標のdouble型2種(lat, lon)を引数にとり、  
変換されたECEF座標がdouble型配列で戻り値として得られる。
#### ECEFtoWGS(double[][])
ECEF座標を測地座標に変換する。  
ECEF座標が保持されたdouble型配列(X,Y,Z)を引数にとり、  
変換された測地座標がdouble型配列(lat,lon,height)で戻り値として得られる。  
#### getENU_Err(Matrix, double, double)  
船舶動向予測技術によって予測された船舶位置とAISデータの位置との距離を算出する。  
ENU座標上にてAISの位置が表されるMatrix型(e,n,u)と、同じくENU座標上で予測位置が表されるdouble型2種(e,n)を引数にとり、  
これらの距離がdouble型で戻り値として得られる。  
#### multi_mat(double[][], double[][])  
行列の積を行うメソッド  
行列を表すdouble型配列2種類を引数にとり  
これらの積がdouble型で戻り値として得られる。  

==========================
### SHIP.java  
SHIPオブジェクトの作成を行う。  
オブジェクトには日付け、時刻、SOG、緯度、経度、COGが関連付けられる。  
#### getSec(String)  
オブジェクトに関連付けられる時刻を秒単位に変換する
AIS受信時刻を示すString型を引数にとり、
この時刻を同日0時からの秒単位に変換した値がint型で戻り値として得られる。 

==========================
### Variance.java
それぞれのの予測時間1秒～3600秒における予測位置とAIS位置の距離の分散を計算する。
各予測時間における平均距離算出後に利用する。  
#### culcVA(double, double)  
誤差距離と平均値を表すdouble型2種類を引数にとり
これらの差の2乗がdouble型で戻り値として得られる。

==========================
### Liner_pre.java
data_sortによって分割、ソートされたAISデータ一ヶ月分を読み込み、  
予測位置と予測時間のAIS位置の距離の平均及び分散を求めるプログラム。
まず原点とするAISデータからtステップ先のAISデータまでの時間差をgetSec()によって取得する。
尚、原点とするAISデータとtステップ先のAISデータが日付をまたぐ際には原点とするAISデータのステップ数を進めるものとした。  
また、時間差が3600秒を超える場合も原点とするAISデータのステップ数を進めるものとした。
次に、原点とするAISデータのSOG,COGを基に時間差の積から予測位置を求めtステップ先のAIS位置との距離の取得を  
get_ENU_Errで行う。  
取得された距離を予測時間に対応する配列total_errに足しこみ次の予測に移る。  
予測原点とするAISデータが任意の回数読み込まれた時点で終了とする。
尚、終了条件となる任意数は便宜的に決定したものである。
#### writeCSV(double, int, double)  
予測された1秒から3600秒後それぞれの予測位置とAIS位置の距離の  
差の総和と、計算回数及び予測時間をCSVファイルに書き込む。  
尚、書きこまれる距離の総和はkm単位である。  
#### writeVar(double, int, int)
予測された1秒から3600秒後それぞれの予測位置とAIS位置の距離の2乗誤差の総和と
計算回数及び予測時間をCSVファイルに書き込む。

============================

## kalman  
カルマンフィルタから得られる状態推定値を用いた線形予測を行う。  
状態推定を行うパラメータには位置・船速を用いたもの、または位置のみを用いたものが存在する。

===========================
### Kalman_pre.java
data_sortによって分割、ソートされたAISデータ一ヶ月分を読み込み、  
カルマンフィルタから得た状態推定値を利用した予測位置と予測時間のAIS位置の距離の平均及び分散を求めるプログラム。  
本プログラムでは状態推定値に位置・船速を用いている。  
まずカルマンフィルタで利用する各パラメータ及び初期値の設定を行う。  
次に予測原点となるjステップ目の状態推定した時間から（j+k）ステップ目のAISデータまでの時間差を  
getSec()によって取得する。  
尚、日付け及び時間差を用いたステップ数を進める条件はLiner_pre.javaと同様にした。  
jステップ時の1秒から3600秒までの予測が終了すると、次にカルマンフィルタによって  
(j+1)ステップ時の状態推定値が計算される。  
カルマンフィルタ予測ステップはPredict()で1秒毎に繰り返し行われる。  
予測ステップでは予測原点からの予測位置が計算されている。  
カルマンフィルタ観測値はm.set()で取得される。  
観測値は予測原点から(j+1)ステップ目のAISデータの位置までの距離をreal_enu_measuredから取得している。  
更新ステップではCorrect()によって予測値と観測値から状態推定値が計算される。  
状態推定された位置成分はjステップ時の位置を原点とした(j+1)ステップ時の位置をENU座標上で表したものである。  
従って(j+1)ステップ時の位置を次の予測原点とするためにECEF座標及び測地座標に戻す必要がある。   
これらの処理をBacktoECEF(),ECEFtoWGS()で行う。  
観測値を計算するために利用するAISデータが任意の回数読み込まれた時点で終了とする。  
尚、終了条件となる任意数は便宜的に決定したものである。  
また予測の誤差の総和、及び誤差平均と予測の2乗誤差の総和の出力はLiner_preと同様である。  

================================
### kalman_only_pos.java  
カルマンフィルタを用いて状態推定するパラメータに位置のみを使用したもの  

================================  
## jkalman  
カルマンフィルタを利用する上でのJavaライブラリ  
(http://jkalman.sourceforge.net/)  

================================  
## jama
行列計算のためのJavaライブラリ  
(http://math.nist.gov/javanumerics/jama/)  

===============================
## jFreeChart  
航跡図プロットのためのグラフ作成ライブラリ  
(http://www.jfree.org/jfreechart/)  

===============================
## steepest_descent_method  
最急降下法によってカルマンフィルタ設定パラメータを決定する  

===============================  
### steepUsePos.java  
kalman_only_pos.java中に最急降下法を追加し、誤差共分散行列を決定した。  
微分刻み幅partial_h及び最急降下法パラメータαを適宜変更し、各偏微分値のノルムがゼロ  
となる誤差共分散行列を探す。  
偏微分値のノルムがゼロとなるときに、手動でプログラムを停止する方法をとった。  

 ==============================
 ### steepUsePos_Sp.java  
 kalman_pre.java中に最急降下法を追加し、誤差共分散行列を決定した。
 状態推定値に位置・速度を用いる以外はsteepUse_pos.javaと同様である。  
 
