## Optimizing std::min_element/argmin in libcxx

### Figures:
When vectorization is **not** enabled:
- Upto 3x speedup (in cycles)
- Outlier: *long long* and *unsigned long long* had a 50% regression.
<img src="figure_novec.png">

When vectorization is enabled:
- Upto 17x speedup (in cycles)
<img src="figure_vec.png">

### Numbers:
The benchmark result files generated from google_benchmark are in the repo:
- `min_element_before.json`
- `min_element_after.json`: performance after using the new algorithm.
- `min_element_after_novec.json`: performance after using the new algorithm with vectorization

Comparing the results using the `compare.py` provided by google_benchmark-

#### Comparing min_element_before.json to min_element_after_novec.json
```
Benchmark                                                                Time             CPU      Time Old      Time New       CPU Old       CPU New
-----------------------------------------------------------------------------------------------------------------------------------------------------
BM_stdmin_element_decreasing<char>/1                                  +0.0101         +0.0102             1             1             1             1
BM_stdmin_element_decreasing<char>/2                                  -0.0898         -0.0898             1             1             1             1
BM_stdmin_element_decreasing<char>/3                                  +0.0103         +0.0102             2             2             2             2
BM_stdmin_element_decreasing<char>/4                                  -0.0516         -0.0516             2             2             2             2
BM_stdmin_element_decreasing<char>/5                                  -0.0077         -0.0077             2             2             2             2
BM_stdmin_element_decreasing<char>/6                                  -0.0494         -0.0494             3             3             3             3
BM_stdmin_element_decreasing<char>/7                                  -0.0647         -0.0647             3             3             3             3
BM_stdmin_element_decreasing<char>/8                                  -0.0447         -0.0448             4             4             4             4
BM_stdmin_element_decreasing<char>/32                                 -0.0913         -0.0913            13            12            13            12
BM_stdmin_element_decreasing<char>/64                                 -0.0312         -0.0312            28            27            28            27
BM_stdmin_element_decreasing<char>/128                                -0.0188         -0.0188            58            57            58            57
BM_stdmin_element_decreasing<char>/256                                -0.1312         -0.1313           118           103           118           103
BM_stdmin_element_decreasing<char>/512                                -0.4212         -0.4212           239           138           239           138
BM_stdmin_element_decreasing<char>/4096                               -0.6690         -0.6690          1931           639          1931           639
BM_stdmin_element_decreasing<char>/5000                               -0.6740         -0.6740          2358           769          2358           769
BM_stdmin_element_decreasing<char>/6000                               -0.6793         -0.6793          2830           907          2830           907
BM_stdmin_element_decreasing<char>/7000                               -0.6830         -0.6830          3301          1047          3301          1047
BM_stdmin_element_decreasing<char>/8000                               -0.6858         -0.6858          3773          1186          3773          1186
BM_stdmin_element_decreasing<char>/9000                               -0.6879         -0.6879          4245          1325          4245          1325
BM_stdmin_element_decreasing<char>/10000                              -0.6899         -0.6899          4717          1463          4717          1463
BM_stdmin_element_decreasing<char>/16384                              -0.6963         -0.6963          7728          2347          7728          2347
BM_stdmin_element_decreasing<char>/32768                              -0.7019         -0.7019         15498          4619         15497          4619
BM_stdmin_element_decreasing<char>/65536                              -0.7035         -0.7035         30987          9189         30987          9188
BM_stdmin_element_decreasing<char>/70000                              -0.7033         -0.7033         33027          9799         33027          9799
BM_stdmin_element_decreasing<short>/1                                 +0.2154         +0.2153             1             1             1             1
BM_stdmin_element_decreasing<short>/2                                 +0.0463         +0.0463             1             1             1             1
BM_stdmin_element_decreasing<short>/3                                 +0.0065         +0.0064             1             1             1             1
BM_stdmin_element_decreasing<short>/4                                 +0.0715         +0.0715             2             2             2             2
BM_stdmin_element_decreasing<short>/5                                 +0.0078         +0.0078             2             2             2             2
BM_stdmin_element_decreasing<short>/6                                 +0.0097         +0.0097             2             2             2             2
BM_stdmin_element_decreasing<short>/7                                 +0.0094         +0.0094             3             3             3             3
BM_stdmin_element_decreasing<short>/8                                 +0.0421         +0.0421             3             3             3             3
BM_stdmin_element_decreasing<short>/32                                -0.0196         -0.0196            12            12            12            12
BM_stdmin_element_decreasing<short>/64                                +0.0038         +0.0038            26            26            26            26
BM_stdmin_element_decreasing<short>/128                               +0.6493         +0.6493            60            99            60            99
BM_stdmin_element_decreasing<short>/256                               -0.1521         -0.1521           120           102           120           102
BM_stdmin_element_decreasing<short>/512                               -0.4351         -0.4351           241           136           241           136
BM_stdmin_element_decreasing<short>/4096                              -0.6916         -0.6916          1932           596          1932           596
BM_stdmin_element_decreasing<short>/5000                              -0.6899         -0.6899          2358           731          2358           731
BM_stdmin_element_decreasing<short>/6000                              -0.7008         -0.7008          2830           847          2830           847
BM_stdmin_element_decreasing<short>/7000                              -0.7183         -0.7183          3401           958          3401           958
BM_stdmin_element_decreasing<short>/8000                              -0.7161         -0.7161          3774          1072          3774          1072
BM_stdmin_element_decreasing<short>/9000                              -0.7219         -0.7221          4245          1180          4245          1180
BM_stdmin_element_decreasing<short>/10000                             -0.7262         -0.7262          4720          1292          4721          1292
BM_stdmin_element_decreasing<short>/16384                             -0.7199         -0.7199          7736          2167          7736          2167
BM_stdmin_element_decreasing<short>/32768                             -0.7239         -0.7239         15464          4270         15463          4270
BM_stdmin_element_decreasing<short>/65536                             -0.7274         -0.7274         31009          8452         31007          8452
BM_stdmin_element_decreasing<short>/70000                             -0.7251         -0.7252         33103          9099         33102          9097
BM_stdmin_element_decreasing<int>/1                                   -0.2449         -0.2449             1             1             1             1
BM_stdmin_element_decreasing<int>/2                                   -0.1019         -0.1019             1             1             1             1
BM_stdmin_element_decreasing<int>/3                                   -0.0777         -0.0777             1             1             1             1
BM_stdmin_element_decreasing<int>/4                                   -0.0469         -0.0473             2             2             2             2
BM_stdmin_element_decreasing<int>/5                                   -0.0477         -0.0479             2             2             2             2
BM_stdmin_element_decreasing<int>/6                                   -0.0415         -0.0416             2             2             2             2
BM_stdmin_element_decreasing<int>/7                                   -0.0366         -0.0368             3             3             3             3
BM_stdmin_element_decreasing<int>/8                                   -0.0320         -0.0320             3             3             3             3
BM_stdmin_element_decreasing<int>/32                                  -0.0188         -0.0190            12            12            12            12
BM_stdmin_element_decreasing<int>/64                                  -0.0124         -0.0125            27            26            27            26
BM_stdmin_element_decreasing<int>/128                                 +0.6450         +0.6445            60            99            60            99
BM_stdmin_element_decreasing<int>/256                                 -0.1580         -0.1580           120           101           120           101
BM_stdmin_element_decreasing<int>/512                                 -0.4403         -0.4403           241           135           241           135
BM_stdmin_element_decreasing<int>/4096                                -0.6866         -0.6867          1932           605          1932           605
BM_stdmin_element_decreasing<int>/5000                                -0.6880         -0.6880          2358           736          2358           736
BM_stdmin_element_decreasing<int>/6000                                -0.7009         -0.7009          2832           847          2831           847
BM_stdmin_element_decreasing<int>/7000                                -0.7330         -0.7327          3604           962          3600           962
BM_stdmin_element_decreasing<int>/8000                                -0.7191         -0.7189          3863          1085          3861          1085
BM_stdmin_element_decreasing<int>/9000                                -0.7100         -0.7100          4345          1260          4345          1260
BM_stdmin_element_decreasing<int>/10000                               -0.7217         -0.7216          4914          1368          4911          1367
BM_stdmin_element_decreasing<int>/16384                               -0.6991         -0.6992          7757          2334          7756          2333
BM_stdmin_element_decreasing<int>/32768                               -0.7097         -0.7095         15501          4500         15489          4500
BM_stdmin_element_decreasing<int>/65536                               -0.7112         -0.7112         30991          8951         30990          8951
BM_stdmin_element_decreasing<int>/70000                               -0.6943         -0.6943         33081         10112         33080         10112
BM_stdmin_element_decreasing<long long>/1                             +0.1993         +0.1994             1             1             1             1
BM_stdmin_element_decreasing<long long>/2                             +0.0125         +0.0104             1             1             1             1
BM_stdmin_element_decreasing<long long>/3                             +0.0109         +0.0110             1             1             1             1
BM_stdmin_element_decreasing<long long>/4                             +0.1808         +0.1807             2             2             2             2
BM_stdmin_element_decreasing<long long>/5                             -0.0750         -0.0773             2             2             2             2
BM_stdmin_element_decreasing<long long>/6                             +0.0001         -0.0011             2             2             2             2
BM_stdmin_element_decreasing<long long>/7                             -0.0368         -0.0367             3             3             3             3
BM_stdmin_element_decreasing<long long>/8                             -0.0109         -0.0108             3             3             3             3
BM_stdmin_element_decreasing<long long>/32                            -0.0230         -0.0238            12            12            12            12
BM_stdmin_element_decreasing<long long>/64                            -0.0077         -0.0077            27            27            27            27
BM_stdmin_element_decreasing<long long>/128                           +0.6288         +0.6288            61            99            61            99
BM_stdmin_element_decreasing<long long>/256                           -0.1644         -0.1644           122           102           122           102
BM_stdmin_element_decreasing<long long>/512                           -0.4538         -0.4537           247           135           247           135
BM_stdmin_element_decreasing<long long>/4096                          -0.6880         -0.6880          1961           612          1961           612
BM_stdmin_element_decreasing<long long>/5000                          -0.6479         -0.6479          2387           841          2387           840
BM_stdmin_element_decreasing<long long>/6000                          -0.6612         -0.6620          2884           977          2884           975
BM_stdmin_element_decreasing<long long>/7000                          -0.6713         -0.6713          3393          1115          3393          1115
BM_stdmin_element_decreasing<long long>/8000                          -0.6767         -0.6776          3836          1240          3836          1237
BM_stdmin_element_decreasing<long long>/9000                          -0.6797         -0.6803          4298          1377          4298          1374
BM_stdmin_element_decreasing<long long>/10000                         -0.6878         -0.6878          4810          1502          4809          1502
BM_stdmin_element_decreasing<long long>/16384                         -0.6811         -0.6811          7895          2517          7894          2517
BM_stdmin_element_decreasing<long long>/32768                         -0.6955         -0.6955         16330          4973         16329          4972
BM_stdmin_element_decreasing<long long>/65536                         -0.6851         -0.6851         32577         10260         32577         10259
BM_stdmin_element_decreasing<long long>/70000                         -0.6836         -0.6836         35316         11174         35313         11174
BM_stdmin_element_decreasing<unsigned char>/1                         +0.0079         +0.0057             1             1             1             1
BM_stdmin_element_decreasing<unsigned char>/2                         -0.1712         -0.1712             1             1             1             1
BM_stdmin_element_decreasing<unsigned char>/3                         -0.1333         -0.1333             2             2             2             2
BM_stdmin_element_decreasing<unsigned char>/4                         -0.1072         -0.1072             2             2             2             2
BM_stdmin_element_decreasing<unsigned char>/5                         -0.0950         -0.0950             3             2             3             2
BM_stdmin_element_decreasing<unsigned char>/6                         +0.0074         +0.0075             3             3             3             3
BM_stdmin_element_decreasing<unsigned char>/7                         -0.0015         -0.0015             3             3             3             3
BM_stdmin_element_decreasing<unsigned char>/8                         +0.1430         +0.1404             4             4             4             4
BM_stdmin_element_decreasing<unsigned char>/32                        -0.0208         -0.0245            12            12            12            12
BM_stdmin_element_decreasing<unsigned char>/64                        -0.2233         -0.2241            35            27            35            27
BM_stdmin_element_decreasing<unsigned char>/128                       -0.1955         -0.1955            73            59            73            59
BM_stdmin_element_decreasing<unsigned char>/256                       -0.2968         -0.2968           149           105           149           105
BM_stdmin_element_decreasing<unsigned char>/512                       -0.5334         -0.5334           301           140           301           140
BM_stdmin_element_decreasing<unsigned char>/4096                      -0.7328         -0.7328          2433           650          2433           650
BM_stdmin_element_decreasing<unsigned char>/5000                      -0.7389         -0.7389          2970           776          2970           775
BM_stdmin_element_decreasing<unsigned char>/6000                      -0.7423         -0.7423          3570           920          3570           920
BM_stdmin_element_decreasing<unsigned char>/7000                      -0.7469         -0.7469          4168          1055          4167          1055
BM_stdmin_element_decreasing<unsigned char>/8000                      -0.6878         -0.6878          3783          1181          3783          1181
BM_stdmin_element_decreasing<unsigned char>/9000                      -0.6898         -0.6898          4258          1321          4258          1321
BM_stdmin_element_decreasing<unsigned char>/10000                     -0.6917         -0.6917          4718          1454          4718          1454
BM_stdmin_element_decreasing<unsigned char>/16384                     -0.6950         -0.6950          7730          2358          7730          2358
BM_stdmin_element_decreasing<unsigned char>/32768                     -0.6999         -0.6999         15522          4658         15522          4658
BM_stdmin_element_decreasing<unsigned char>/65536                     -0.7031         -0.7031         31153          9248         31153          9248
BM_stdmin_element_decreasing<unsigned char>/70000                     -0.7015         -0.7032         33251          9924         33251          9869
BM_stdmin_element_decreasing<unsigned short>/1                        -0.2428         -0.2436             1             1             1             1
BM_stdmin_element_decreasing<unsigned short>/2                        -0.0936         -0.0936             1             1             1             1
BM_stdmin_element_decreasing<unsigned short>/3                        +0.0110         +0.0110             1             1             1             1
BM_stdmin_element_decreasing<unsigned short>/4                        +0.0532         +0.0533             2             2             2             2
BM_stdmin_element_decreasing<unsigned short>/5                        +0.1262         +0.1261             2             2             2             2
BM_stdmin_element_decreasing<unsigned short>/6                        +0.1567         +0.1568             2             3             2             3
BM_stdmin_element_decreasing<unsigned short>/7                        +0.1807         +0.1807             3             3             3             3
BM_stdmin_element_decreasing<unsigned short>/8                        +0.1813         +0.1812             3             4             3             4
BM_stdmin_element_decreasing<unsigned short>/32                       +0.3230         +0.3230            12            16            12            16
BM_stdmin_element_decreasing<unsigned short>/64                       +0.1713         +0.1712            27            31            27            31
BM_stdmin_element_decreasing<unsigned short>/128                      +0.6497         +0.6497            60            99            60            99
BM_stdmin_element_decreasing<unsigned short>/256                      -0.1553         -0.1553           120           102           120           102
BM_stdmin_element_decreasing<unsigned short>/512                      -0.4447         -0.4447           242           135           242           135
BM_stdmin_element_decreasing<unsigned short>/4096                     -0.6933         -0.6933          1964           602          1964           602
BM_stdmin_element_decreasing<unsigned short>/5000                     -0.6866         -0.6866          2363           740          2363           740
BM_stdmin_element_decreasing<unsigned short>/6000                     -0.7026         -0.7026          2857           850          2857           850
BM_stdmin_element_decreasing<unsigned short>/7000                     -0.7057         -0.7057          3305           973          3305           973
BM_stdmin_element_decreasing<unsigned short>/8000                     -0.7150         -0.7150          3776          1076          3776          1076
BM_stdmin_element_decreasing<unsigned short>/9000                     -0.7204         -0.7204          4247          1187          4247          1187
BM_stdmin_element_decreasing<unsigned short>/10000                    -0.7247         -0.7247          4721          1300          4721          1300
BM_stdmin_element_decreasing<unsigned short>/16384                    -0.7166         -0.7166          7744          2195          7743          2194
BM_stdmin_element_decreasing<unsigned short>/32768                    -0.7124         -0.7124         15465          4448         15465          4448
BM_stdmin_element_decreasing<unsigned short>/65536                    -0.7204         -0.7204         30958          8656         30959          8656
BM_stdmin_element_decreasing<unsigned short>/70000                    -0.7210         -0.7210         33395          9318         33396          9317
BM_stdmin_element_decreasing<unsigned int>/1                          -0.2270         -0.2270             1             1             1             1
BM_stdmin_element_decreasing<unsigned int>/2                          -0.1838         -0.1839             1             1             1             1
BM_stdmin_element_decreasing<unsigned int>/3                          -0.0719         -0.0719             1             1             1             1
BM_stdmin_element_decreasing<unsigned int>/4                          -0.0549         -0.0549             2             2             2             2
BM_stdmin_element_decreasing<unsigned int>/5                          -0.0436         -0.0435             2             2             2             2
BM_stdmin_element_decreasing<unsigned int>/6                          -0.0708         -0.0707             3             2             3             2
BM_stdmin_element_decreasing<unsigned int>/7                          -0.0194         -0.0238             3             3             3             3
BM_stdmin_element_decreasing<unsigned int>/8                          -0.0601         -0.0601             3             3             3             3
BM_stdmin_element_decreasing<unsigned int>/32                         -0.0145         -0.0145            12            12            12            12
BM_stdmin_element_decreasing<unsigned int>/64                         -0.0070         -0.0069            27            27            27            27
BM_stdmin_element_decreasing<unsigned int>/128                        +0.6668         +0.6667            61           101            61           101
BM_stdmin_element_decreasing<unsigned int>/256                        -0.1316         -0.1316           121           105           121           105
BM_stdmin_element_decreasing<unsigned int>/512                        -0.4289         -0.4289           241           138           241           138
BM_stdmin_element_decreasing<unsigned int>/4096                       -0.6840         -0.6840          1933           611          1933           611
BM_stdmin_element_decreasing<unsigned int>/5000                       -0.6848         -0.6848          2359           744          2359           744
BM_stdmin_element_decreasing<unsigned int>/6000                       -0.6988         -0.6989          2832           853          2832           853
BM_stdmin_element_decreasing<unsigned int>/7000                       -0.7079         -0.7079          3302           965          3302           965
BM_stdmin_element_decreasing<unsigned int>/8000                       -0.7135         -0.7135          3776          1082          3776          1082
BM_stdmin_element_decreasing<unsigned int>/9000                       -0.7072         -0.7072          4251          1245          4251          1245
BM_stdmin_element_decreasing<unsigned int>/10000                      -0.7111         -0.7111          4722          1364          4722          1364
BM_stdmin_element_decreasing<unsigned int>/16384                      -0.7123         -0.7123          7911          2276          7911          2276
BM_stdmin_element_decreasing<unsigned int>/32768                      -0.7129         -0.7129         15576          4472         15576          4472
BM_stdmin_element_decreasing<unsigned int>/65536                      -0.7133         -0.7134         30982          8881         30982          8881
BM_stdmin_element_decreasing<unsigned int>/70000                      -0.7064         -0.7064         33058          9705         33058          9705
BM_stdmin_element_decreasing<unsigned long long>/1                    +0.2137         +0.2136             1             1             1             1
BM_stdmin_element_decreasing<unsigned long long>/2                    +0.2645         +0.2645             1             1             1             1
BM_stdmin_element_decreasing<unsigned long long>/3                    +0.1062         +0.1062             1             1             1             1
BM_stdmin_element_decreasing<unsigned long long>/4                    +0.1522         +0.1522             2             2             2             2
BM_stdmin_element_decreasing<unsigned long long>/5                    +0.0714         +0.0714             2             2             2             2
BM_stdmin_element_decreasing<unsigned long long>/6                    +0.1123         +0.1123             2             3             2             3
BM_stdmin_element_decreasing<unsigned long long>/7                    +0.0550         +0.0550             3             3             3             3
BM_stdmin_element_decreasing<unsigned long long>/8                    +0.1213         +0.1213             3             3             3             3
BM_stdmin_element_decreasing<unsigned long long>/32                   +0.0175         +0.0175            12            12            12            12
BM_stdmin_element_decreasing<unsigned long long>/64                   +0.0128         +0.0128            27            27            27            27
BM_stdmin_element_decreasing<unsigned long long>/128                  +0.7018         +0.7018            60           102            60           102
BM_stdmin_element_decreasing<unsigned long long>/256                  -0.1452         -0.1452           120           103           120           103
BM_stdmin_element_decreasing<unsigned long long>/512                  -0.4386         -0.4386           241           135           241           135
BM_stdmin_element_decreasing<unsigned long long>/4096                 -0.6850         -0.6850          1936           610          1936           610
BM_stdmin_element_decreasing<unsigned long long>/5000                 -0.6477         -0.6477          2364           833          2364           833
BM_stdmin_element_decreasing<unsigned long long>/6000                 -0.6581         -0.6581          2835           969          2835           969
BM_stdmin_element_decreasing<unsigned long long>/7000                 -0.6657         -0.6657          3307          1106          3307          1106
BM_stdmin_element_decreasing<unsigned long long>/8000                 -0.6761         -0.6762          3778          1224          3778          1223
BM_stdmin_element_decreasing<unsigned long long>/9000                 -0.6733         -0.6733          4251          1389          4251          1389
BM_stdmin_element_decreasing<unsigned long long>/10000                -0.6809         -0.6809          4723          1507          4723          1507
BM_stdmin_element_decreasing<unsigned long long>/16384                -0.6727         -0.6728          7737          2532          7737          2532
BM_stdmin_element_decreasing<unsigned long long>/32768                -0.6789         -0.6789         15478          4969         15478          4969
BM_stdmin_element_decreasing<unsigned long long>/65536                -0.6531         -0.6531         31046         10769         31046         10769
BM_stdmin_element_decreasing<unsigned long long>/70000                -0.6554         -0.6554         33148         11424         33147         11424
OVERALL_GEOMEAN                                                       -0.4341         -0.4342             0             0             0             0
```

#### Comparing min_element_before.json to min_element_after.json

```
Benchmark                                                                Time             CPU      Time Old      Time New       CPU Old       CPU New
-----------------------------------------------------------------------------------------------------------------------------------------------------
BM_stdmin_element_decreasing<char>/1                                  +0.0005         +0.0006             1             1             1             1
BM_stdmin_element_decreasing<char>/2                                  -0.0982         -0.0981             1             1             1             1
BM_stdmin_element_decreasing<char>/3                                  +0.0012         +0.0011             2             2             2             2
BM_stdmin_element_decreasing<char>/4                                  -0.0491         -0.0491             2             2             2             2
BM_stdmin_element_decreasing<char>/5                                  -0.0163         -0.0163             2             2             2             2
BM_stdmin_element_decreasing<char>/6                                  -0.0801         -0.0800             3             3             3             3
BM_stdmin_element_decreasing<char>/7                                  -0.0678         -0.0678             3             3             3             3
BM_stdmin_element_decreasing<char>/8                                  -0.0631         -0.0631             4             4             4             4
BM_stdmin_element_decreasing<char>/32                                 -0.0987         -0.0988            13            12            13            12
BM_stdmin_element_decreasing<char>/64                                 -0.0342         -0.0342            28            27            28            27
BM_stdmin_element_decreasing<char>/128                                -0.2086         -0.2086            58            46            58            46
BM_stdmin_element_decreasing<char>/256                                -0.3517         -0.3517           118            77           118            77
BM_stdmin_element_decreasing<char>/512                                -0.6520         -0.6520           239            83           239            83
BM_stdmin_element_decreasing<char>/4096                               -0.9017         -0.9017          1931           190          1931           190
BM_stdmin_element_decreasing<char>/5000                               -0.9054         -0.9054          2358           223          2358           223
BM_stdmin_element_decreasing<char>/6000                               -0.9128         -0.9128          2830           247          2830           247
BM_stdmin_element_decreasing<char>/7000                               -0.9145         -0.9145          3301           282          3301           282
BM_stdmin_element_decreasing<char>/8000                               -0.9190         -0.9190          3773           306          3773           306
BM_stdmin_element_decreasing<char>/9000                               -0.9193         -0.9193          4245           343          4245           343
BM_stdmin_element_decreasing<char>/10000                              -0.9223         -0.9223          4717           367          4717           367
BM_stdmin_element_decreasing<char>/16384                              -0.9287         -0.9287          7728           551          7728           551
BM_stdmin_element_decreasing<char>/32768                              -0.9332         -0.9332         15498          1036         15497          1036
BM_stdmin_element_decreasing<char>/65536                              -0.9354         -0.9354         30987          2001         30987          2001
BM_stdmin_element_decreasing<char>/70000                              -0.9354         -0.9354         33027          2133         33027          2133
BM_stdmin_element_decreasing<short>/1                                 +0.2019         +0.2019             1             1             1             1
BM_stdmin_element_decreasing<short>/2                                 +0.0009         +0.0009             1             1             1             1
BM_stdmin_element_decreasing<short>/3                                 -0.0194         -0.0194             1             1             1             1
BM_stdmin_element_decreasing<short>/4                                 +0.1333         +0.1333             2             2             2             2
BM_stdmin_element_decreasing<short>/5                                 +0.0075         +0.0075             2             2             2             2
BM_stdmin_element_decreasing<short>/6                                 +0.0003         +0.0004             2             2             2             2
BM_stdmin_element_decreasing<short>/7                                 +0.0009         +0.0009             3             3             3             3
BM_stdmin_element_decreasing<short>/8                                 +0.0241         +0.0240             3             3             3             3
BM_stdmin_element_decreasing<short>/32                                -0.0049         -0.0049            12            12            12            12
BM_stdmin_element_decreasing<short>/64                                -0.0071         -0.0071            26            26            26            26
BM_stdmin_element_decreasing<short>/128                               +0.1354         +0.1353            60            68            60            68
BM_stdmin_element_decreasing<short>/256                               +0.0684         +0.0684           120           128           120           128
BM_stdmin_element_decreasing<short>/512                               -0.4243         -0.4243           241           139           241           139
BM_stdmin_element_decreasing<short>/4096                              -0.8732         -0.8732          1932           245          1932           245
BM_stdmin_element_decreasing<short>/5000                              -0.9075         -0.9075          2358           218          2358           218
BM_stdmin_element_decreasing<short>/6000                              -0.9154         -0.9154          2830           239          2830           239
BM_stdmin_element_decreasing<short>/7000                              -0.9243         -0.9243          3401           258          3401           258
BM_stdmin_element_decreasing<short>/8000                              -0.9277         -0.9277          3774           273          3774           273
BM_stdmin_element_decreasing<short>/9000                              -0.9327         -0.9327          4245           286          4245           286
BM_stdmin_element_decreasing<short>/10000                             -0.9390         -0.9390          4720           288          4721           288
BM_stdmin_element_decreasing<short>/16384                             -0.9205         -0.9205          7736           615          7736           615
BM_stdmin_element_decreasing<short>/32768                             -0.9282         -0.9283         15464          1110         15463          1109
BM_stdmin_element_decreasing<short>/65536                             -0.9325         -0.9325         31009          2092         31007          2092
BM_stdmin_element_decreasing<short>/70000                             -0.9325         -0.9325         33103          2233         33102          2233
BM_stdmin_element_decreasing<int>/1                                   +0.0019         +0.0019             1             1             1             1
BM_stdmin_element_decreasing<int>/2                                   +0.1120         +0.1120             1             1             1             1
BM_stdmin_element_decreasing<int>/3                                   -0.0012         -0.0012             1             1             1             1
BM_stdmin_element_decreasing<int>/4                                   +0.0967         +0.0966             2             2             2             2
BM_stdmin_element_decreasing<int>/5                                   +0.0009         +0.0009             2             2             2             2
BM_stdmin_element_decreasing<int>/6                                   +0.0468         +0.0468             2             3             2             3
BM_stdmin_element_decreasing<int>/7                                   -0.0011         -0.0010             3             3             3             3
BM_stdmin_element_decreasing<int>/8                                   +0.0362         +0.0361             3             3             3             3
BM_stdmin_element_decreasing<int>/32                                  +0.0069         +0.0069            12            12            12            12
BM_stdmin_element_decreasing<int>/64                                  +0.0005         +0.0005            27            27            27            27
BM_stdmin_element_decreasing<int>/128                                 -0.2361         -0.2361            60            46            60            46
BM_stdmin_element_decreasing<int>/256                                 -0.3366         -0.3366           120            80           120            80
BM_stdmin_element_decreasing<int>/512                                 -0.6283         -0.6284           241            90           241            90
BM_stdmin_element_decreasing<int>/4096                                -0.8909         -0.8909          1932           211          1932           211
BM_stdmin_element_decreasing<int>/5000                                -0.9070         -0.9070          2358           219          2358           219
BM_stdmin_element_decreasing<int>/6000                                -0.9108         -0.9108          2832           253          2831           253
BM_stdmin_element_decreasing<int>/7000                                -0.9203         -0.9203          3604           287          3600           287
BM_stdmin_element_decreasing<int>/8000                                -0.9194         -0.9193          3863           311          3861           311
BM_stdmin_element_decreasing<int>/9000                                -0.9215         -0.9215          4345           341          4345           341
BM_stdmin_element_decreasing<int>/10000                               -0.9260         -0.9259          4914           364          4911           364
BM_stdmin_element_decreasing<int>/16384                               -0.9165         -0.9165          7757           647          7756           647
BM_stdmin_element_decreasing<int>/32768                               -0.9216         -0.9215         15501          1216         15489          1216
BM_stdmin_element_decreasing<int>/65536                               -0.9237         -0.9237         30991          2363         30990          2363
BM_stdmin_element_decreasing<int>/70000                               -0.9246         -0.9246         33081          2495         33080          2495
BM_stdmin_element_decreasing<long long>/1                             +0.1668         +0.1668             1             1             1             1
BM_stdmin_element_decreasing<long long>/2                             -0.0151         -0.0143             1             1             1             1
BM_stdmin_element_decreasing<long long>/3                             -0.0319         -0.0318             1             1             1             1
BM_stdmin_element_decreasing<long long>/4                             +0.1573         +0.1572             2             2             2             2
BM_stdmin_element_decreasing<long long>/5                             -0.0983         -0.0983             2             2             2             2
BM_stdmin_element_decreasing<long long>/6                             -0.0201         -0.0201             2             2             2             2
BM_stdmin_element_decreasing<long long>/7                             -0.0515         -0.0514             3             3             3             3
BM_stdmin_element_decreasing<long long>/8                             -0.0257         -0.0257             3             3             3             3
BM_stdmin_element_decreasing<long long>/32                            -0.0372         -0.0372            12            12            12            12
BM_stdmin_element_decreasing<long long>/64                            -0.0227         -0.0228            27            26            27            26
BM_stdmin_element_decreasing<long long>/128                           -0.1179         -0.1179            61            54            61            54
BM_stdmin_element_decreasing<long long>/256                           -0.2302         -0.2302           122            94           122            94
BM_stdmin_element_decreasing<long long>/512                           -0.5197         -0.5197           247           119           247           119
BM_stdmin_element_decreasing<long long>/4096                          -0.7574         -0.7575          1961           476          1961           476
BM_stdmin_element_decreasing<long long>/5000                          -0.7719         -0.7719          2387           544          2387           544
BM_stdmin_element_decreasing<long long>/6000                          -0.7798         -0.7798          2884           635          2884           635
BM_stdmin_element_decreasing<long long>/7000                          -0.7841         -0.7841          3393           733          3393           733
BM_stdmin_element_decreasing<long long>/8000                          -0.7859         -0.7859          3836           821          3836           821
BM_stdmin_element_decreasing<long long>/9000                          -0.7873         -0.7873          4298           914          4298           914
BM_stdmin_element_decreasing<long long>/10000                         -0.7935         -0.7934          4810           993          4809           993
BM_stdmin_element_decreasing<long long>/16384                         -0.7857         -0.7857          7895          1692          7894          1692
BM_stdmin_element_decreasing<long long>/32768                         -0.7966         -0.7966         16330          3322         16329          3322
BM_stdmin_element_decreasing<long long>/65536                         -0.7909         -0.7909         32577          6813         32577          6813
BM_stdmin_element_decreasing<long long>/70000                         -0.7920         -0.7920         35316          7344         35313          7344
BM_stdmin_element_decreasing<unsigned char>/1                         -0.0188         -0.0187             1             1             1             1
BM_stdmin_element_decreasing<unsigned char>/2                         -0.1893         -0.1893             1             1             1             1
BM_stdmin_element_decreasing<unsigned char>/3                         -0.1535         -0.1535             2             2             2             2
BM_stdmin_element_decreasing<unsigned char>/4                         -0.1244         -0.1244             2             2             2             2
BM_stdmin_element_decreasing<unsigned char>/5                         -0.1129         -0.1129             3             2             3             2
BM_stdmin_element_decreasing<unsigned char>/6                         -0.0588         -0.0588             3             3             3             3
BM_stdmin_element_decreasing<unsigned char>/7                         -0.0804         -0.0804             3             3             3             3
BM_stdmin_element_decreasing<unsigned char>/8                         -0.0563         -0.0563             4             3             4             3
BM_stdmin_element_decreasing<unsigned char>/32                        -0.0578         -0.0578            12            12            12            12
BM_stdmin_element_decreasing<unsigned char>/64                        -0.2375         -0.2375            35            27            35            27
BM_stdmin_element_decreasing<unsigned char>/128                       -0.4025         -0.4025            73            43            73            43
BM_stdmin_element_decreasing<unsigned char>/256                       -0.4973         -0.4973           149            75           149            75
BM_stdmin_element_decreasing<unsigned char>/512                       -0.7303         -0.7303           301            81           301            81
BM_stdmin_element_decreasing<unsigned char>/4096                      -0.9267         -0.9267          2433           178          2433           178
BM_stdmin_element_decreasing<unsigned char>/5000                      -0.9289         -0.9289          2970           211          2970           211
BM_stdmin_element_decreasing<unsigned char>/6000                      -0.9353         -0.9353          3570           231          3570           231
BM_stdmin_element_decreasing<unsigned char>/7000                      -0.9363         -0.9363          4168           265          4167           265
BM_stdmin_element_decreasing<unsigned char>/8000                      -0.9243         -0.9243          3783           287          3783           287
BM_stdmin_element_decreasing<unsigned char>/9000                      -0.9248         -0.9248          4258           320          4258           320
BM_stdmin_element_decreasing<unsigned char>/10000                     -0.9279         -0.9279          4718           340          4718           340
BM_stdmin_element_decreasing<unsigned char>/16384                     -0.9339         -0.9339          7730           511          7730           511
BM_stdmin_element_decreasing<unsigned char>/32768                     -0.9385         -0.9385         15522           955         15522           955
BM_stdmin_element_decreasing<unsigned char>/65536                     -0.9407         -0.9407         31153          1847         31153          1847
BM_stdmin_element_decreasing<unsigned char>/70000                     -0.9408         -0.9408         33251          1970         33251          1970
BM_stdmin_element_decreasing<unsigned short>/1                        -0.2614         -0.2614             1             1             1             1
BM_stdmin_element_decreasing<unsigned short>/2                        -0.1283         -0.1282             1             1             1             1
BM_stdmin_element_decreasing<unsigned short>/3                        -0.0982         -0.0982             1             1             1             1
BM_stdmin_element_decreasing<unsigned short>/4                        -0.0216         -0.0216             2             2             2             2
BM_stdmin_element_decreasing<unsigned short>/5                        -0.0558         -0.0558             2             2             2             2
BM_stdmin_element_decreasing<unsigned short>/6                        -0.0477         -0.0476             2             2             2             2
BM_stdmin_element_decreasing<unsigned short>/7                        -0.0412         -0.0413             3             3             3             3
BM_stdmin_element_decreasing<unsigned short>/8                        -0.0509         -0.0509             3             3             3             3
BM_stdmin_element_decreasing<unsigned short>/32                       -0.0274         -0.0274            12            12            12            12
BM_stdmin_element_decreasing<unsigned short>/64                       -0.0255         -0.0255            27            26            27            26
BM_stdmin_element_decreasing<unsigned short>/128                      -0.1539         -0.1539            60            51            60            51
BM_stdmin_element_decreasing<unsigned short>/256                      -0.3518         -0.3519           120            78           120            78
BM_stdmin_element_decreasing<unsigned short>/512                      -0.6546         -0.6546           242            84           242            84
BM_stdmin_element_decreasing<unsigned short>/4096                     -0.9071         -0.9071          1964           182          1964           182
BM_stdmin_element_decreasing<unsigned short>/5000                     -0.9224         -0.9224          2363           183          2363           183
BM_stdmin_element_decreasing<unsigned short>/6000                     -0.9271         -0.9271          2857           208          2857           208
BM_stdmin_element_decreasing<unsigned short>/7000                     -0.9301         -0.9301          3305           231          3305           231
BM_stdmin_element_decreasing<unsigned short>/8000                     -0.9340         -0.9340          3776           249          3776           249
BM_stdmin_element_decreasing<unsigned short>/9000                     -0.9380         -0.9380          4247           263          4247           263
BM_stdmin_element_decreasing<unsigned short>/10000                    -0.9417         -0.9417          4721           275          4721           275
BM_stdmin_element_decreasing<unsigned short>/16384                    -0.9322         -0.9322          7744           525          7743           525
BM_stdmin_element_decreasing<unsigned short>/32768                    -0.9364         -0.9364         15465           984         15465           984
BM_stdmin_element_decreasing<unsigned short>/65536                    -0.9388         -0.9388         30958          1895         30959          1895
BM_stdmin_element_decreasing<unsigned short>/70000                    -0.9393         -0.9393         33395          2026         33396          2025
BM_stdmin_element_decreasing<unsigned int>/1                          -0.0067         -0.0067             1             1             1             1
BM_stdmin_element_decreasing<unsigned int>/2                          -0.0090         -0.0089             1             1             1             1
BM_stdmin_element_decreasing<unsigned int>/3                          -0.0007         -0.0008             1             1             1             1
BM_stdmin_element_decreasing<unsigned int>/4                          +0.1938         +0.1938             2             2             2             2
BM_stdmin_element_decreasing<unsigned int>/5                          +0.0002         +0.0002             2             2             2             2
BM_stdmin_element_decreasing<unsigned int>/6                          +0.0011         +0.0012             3             3             3             3
BM_stdmin_element_decreasing<unsigned int>/7                          -0.0002         -0.0002             3             3             3             3
BM_stdmin_element_decreasing<unsigned int>/8                          +0.0015         +0.0014             3             3             3             3
BM_stdmin_element_decreasing<unsigned int>/32                         -0.0033         -0.0033            12            12            12            12
BM_stdmin_element_decreasing<unsigned int>/64                         -0.0102         -0.0102            27            27            27            27
BM_stdmin_element_decreasing<unsigned int>/128                        -0.2116         -0.2116            61            48            61            48
BM_stdmin_element_decreasing<unsigned int>/256                        -0.3256         -0.3256           121            82           121            82
BM_stdmin_element_decreasing<unsigned int>/512                        -0.6368         -0.6368           241            88           241            88
BM_stdmin_element_decreasing<unsigned int>/4096                       -0.8904         -0.8904          1933           212          1933           212
BM_stdmin_element_decreasing<unsigned int>/5000                       -0.9071         -0.9071          2359           219          2359           219
BM_stdmin_element_decreasing<unsigned int>/6000                       -0.9107         -0.9107          2832           253          2832           253
BM_stdmin_element_decreasing<unsigned int>/7000                       -0.9134         -0.9134          3302           286          3302           286
BM_stdmin_element_decreasing<unsigned int>/8000                       -0.9190         -0.9190          3776           306          3776           306
BM_stdmin_element_decreasing<unsigned int>/9000                       -0.9205         -0.9205          4251           338          4251           338
BM_stdmin_element_decreasing<unsigned int>/10000                      -0.9229         -0.9229          4722           364          4722           364
BM_stdmin_element_decreasing<unsigned int>/16384                      -0.9185         -0.9185          7911           645          7911           645
BM_stdmin_element_decreasing<unsigned int>/32768                      -0.9220         -0.9220         15576          1215         15576          1215
BM_stdmin_element_decreasing<unsigned int>/65536                      -0.9238         -0.9238         30982          2360         30982          2359
BM_stdmin_element_decreasing<unsigned int>/70000                      -0.9244         -0.9244         33058          2500         33058          2500
BM_stdmin_element_decreasing<unsigned long long>/1                    +0.2015         +0.2014             1             1             1             1
BM_stdmin_element_decreasing<unsigned long long>/2                    +0.0153         +0.0153             1             1             1             1
BM_stdmin_element_decreasing<unsigned long long>/3                    +0.0114         +0.0114             1             1             1             1
BM_stdmin_element_decreasing<unsigned long long>/4                    +0.0913         +0.0913             2             2             2             2
BM_stdmin_element_decreasing<unsigned long long>/5                    +0.0005         +0.0004             2             2             2             2
BM_stdmin_element_decreasing<unsigned long long>/6                    -0.0000         -0.0000             2             2             2             2
BM_stdmin_element_decreasing<unsigned long long>/7                    -0.0002         -0.0002             3             3             3             3
BM_stdmin_element_decreasing<unsigned long long>/8                    +0.0106         +0.0106             3             3             3             3
BM_stdmin_element_decreasing<unsigned long long>/32                   -0.0323         -0.0323            12            12            12            12
BM_stdmin_element_decreasing<unsigned long long>/64                   -0.0156         -0.0156            27            26            27            26
BM_stdmin_element_decreasing<unsigned long long>/128                  -0.0222         -0.0222            60            59            60            59
BM_stdmin_element_decreasing<unsigned long long>/256                  -0.1703         -0.1703           120           100           120           100
BM_stdmin_element_decreasing<unsigned long long>/512                  -0.4509         -0.4510           241           132           241           132
BM_stdmin_element_decreasing<unsigned long long>/4096                 -0.6958         -0.6958          1936           589          1936           589
BM_stdmin_element_decreasing<unsigned long long>/5000                 -0.7105         -0.7105          2364           684          2364           684
BM_stdmin_element_decreasing<unsigned long long>/6000                 -0.7166         -0.7166          2835           803          2835           803
BM_stdmin_element_decreasing<unsigned long long>/7000                 -0.7192         -0.7192          3307           929          3307           929
BM_stdmin_element_decreasing<unsigned long long>/8000                 -0.7241         -0.7241          3778          1042          3778          1042
BM_stdmin_element_decreasing<unsigned long long>/9000                 -0.7255         -0.7256          4251          1167          4251          1167
BM_stdmin_element_decreasing<unsigned long long>/10000                -0.7295         -0.7295          4723          1277          4723          1277
BM_stdmin_element_decreasing<unsigned long long>/16384                -0.7223         -0.7223          7737          2149          7737          2149
BM_stdmin_element_decreasing<unsigned long long>/32768                -0.7261         -0.7261         15478          4240         15478          4240
BM_stdmin_element_decreasing<unsigned long long>/65536                -0.7264         -0.7264         31046          8493         31046          8493
BM_stdmin_element_decreasing<unsigned long long>/70000                -0.7281         -0.7281         33148          9012         33147          9011
OVERALL_GEOMEAN                                                       -0.6720         -0.6720             0             0             0             0
