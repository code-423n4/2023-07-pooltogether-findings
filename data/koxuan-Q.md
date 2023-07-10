# Report


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1) | Constants in comparisons should appear on the left side | 163 |
| [NC-2](#NC-2) | Lines are too long | 5 |
| [NC-3](#NC-3) | Event is missing `indexed` fields | 7 |
| [NC-4](#NC-4) | Constants should be defined rather than using magic numbers | 98 |
| [NC-5](#NC-5) | Variable names don't follow the Solidity style guide | 2 |
### <a name="NC-1"></a>[NC-1] Constants in comparisons should appear on the left side
Constants should appear on the left side of comparisons, to avoid accidental assignment

*Instances (163)*:
```solidity
File: PrizePool.sol

350:     if (winningRandomNumber_ == 0) {

763:       (lastClosedDrawId == 0 ? 1 : 2) *

896:     if (_lastClosedDrawId == 0) {

```

```solidity
File: abstract/TieredLiquidityDistributor.sol

731:     if (numTiers == 3) {

733:     } else if (numTiers == 4) {

735:     } else if (numTiers == 5) {

737:     } else if (numTiers == 6) {

739:     } else if (numTiers == 7) {

741:     } else if (numTiers == 8) {

743:     } else if (numTiers == 9) {

745:     } else if (numTiers == 10) {

747:     } else if (numTiers == 11) {

749:     } else if (numTiers == 12) {

751:     } else if (numTiers == 13) {

753:     } else if (numTiers == 14) {

755:     } else if (numTiers == 15) {

765:     if (numTiers == 3) {

767:     } else if (numTiers == 4) {

769:     } else if (numTiers == 5) {

771:     } else if (numTiers == 6) {

773:     } else if (numTiers == 7) {

775:     } else if (numTiers == 8) {

777:     } else if (numTiers == 9) {

779:     } else if (numTiers == 10) {

781:     } else if (numTiers == 11) {

783:     } else if (numTiers == 12) {

785:     } else if (numTiers == 13) {

787:     } else if (numTiers == 14) {

789:     } else if (numTiers == 15) {

808:     if (_numTiers == 3) {

809:       if (_tier == 0) return TIER_ODDS_0_3;

810:       else if (_tier == 1) return TIER_ODDS_1_3;

811:       else if (_tier == 2) return TIER_ODDS_2_3;

812:     } else if (_numTiers == 4) {

813:       if (_tier == 0) return TIER_ODDS_0_4;

814:       else if (_tier == 1) return TIER_ODDS_1_4;

815:       else if (_tier == 2) return TIER_ODDS_2_4;

816:       else if (_tier == 3) return TIER_ODDS_3_4;

817:     } else if (_numTiers == 5) {

818:       if (_tier == 0) return TIER_ODDS_0_5;

819:       else if (_tier == 1) return TIER_ODDS_1_5;

820:       else if (_tier == 2) return TIER_ODDS_2_5;

821:       else if (_tier == 3) return TIER_ODDS_3_5;

822:       else if (_tier == 4) return TIER_ODDS_4_5;

823:     } else if (_numTiers == 6) {

824:       if (_tier == 0) return TIER_ODDS_0_6;

825:       else if (_tier == 1) return TIER_ODDS_1_6;

826:       else if (_tier == 2) return TIER_ODDS_2_6;

827:       else if (_tier == 3) return TIER_ODDS_3_6;

828:       else if (_tier == 4) return TIER_ODDS_4_6;

829:       else if (_tier == 5) return TIER_ODDS_5_6;

830:     } else if (_numTiers == 7) {

831:       if (_tier == 0) return TIER_ODDS_0_7;

832:       else if (_tier == 1) return TIER_ODDS_1_7;

833:       else if (_tier == 2) return TIER_ODDS_2_7;

834:       else if (_tier == 3) return TIER_ODDS_3_7;

835:       else if (_tier == 4) return TIER_ODDS_4_7;

836:       else if (_tier == 5) return TIER_ODDS_5_7;

837:       else if (_tier == 6) return TIER_ODDS_6_7;

838:     } else if (_numTiers == 8) {

839:       if (_tier == 0) return TIER_ODDS_0_8;

840:       else if (_tier == 1) return TIER_ODDS_1_8;

841:       else if (_tier == 2) return TIER_ODDS_2_8;

842:       else if (_tier == 3) return TIER_ODDS_3_8;

843:       else if (_tier == 4) return TIER_ODDS_4_8;

844:       else if (_tier == 5) return TIER_ODDS_5_8;

845:       else if (_tier == 6) return TIER_ODDS_6_8;

846:       else if (_tier == 7) return TIER_ODDS_7_8;

847:     } else if (_numTiers == 9) {

848:       if (_tier == 0) return TIER_ODDS_0_9;

849:       else if (_tier == 1) return TIER_ODDS_1_9;

850:       else if (_tier == 2) return TIER_ODDS_2_9;

851:       else if (_tier == 3) return TIER_ODDS_3_9;

852:       else if (_tier == 4) return TIER_ODDS_4_9;

853:       else if (_tier == 5) return TIER_ODDS_5_9;

854:       else if (_tier == 6) return TIER_ODDS_6_9;

855:       else if (_tier == 7) return TIER_ODDS_7_9;

856:       else if (_tier == 8) return TIER_ODDS_8_9;

857:     } else if (_numTiers == 10) {

858:       if (_tier == 0) return TIER_ODDS_0_10;

859:       else if (_tier == 1) return TIER_ODDS_1_10;

860:       else if (_tier == 2) return TIER_ODDS_2_10;

861:       else if (_tier == 3) return TIER_ODDS_3_10;

862:       else if (_tier == 4) return TIER_ODDS_4_10;

863:       else if (_tier == 5) return TIER_ODDS_5_10;

864:       else if (_tier == 6) return TIER_ODDS_6_10;

865:       else if (_tier == 7) return TIER_ODDS_7_10;

866:       else if (_tier == 8) return TIER_ODDS_8_10;

867:       else if (_tier == 9) return TIER_ODDS_9_10;

868:     } else if (_numTiers == 11) {

869:       if (_tier == 0) return TIER_ODDS_0_11;

870:       else if (_tier == 1) return TIER_ODDS_1_11;

871:       else if (_tier == 2) return TIER_ODDS_2_11;

872:       else if (_tier == 3) return TIER_ODDS_3_11;

873:       else if (_tier == 4) return TIER_ODDS_4_11;

874:       else if (_tier == 5) return TIER_ODDS_5_11;

875:       else if (_tier == 6) return TIER_ODDS_6_11;

876:       else if (_tier == 7) return TIER_ODDS_7_11;

877:       else if (_tier == 8) return TIER_ODDS_8_11;

878:       else if (_tier == 9) return TIER_ODDS_9_11;

879:       else if (_tier == 10) return TIER_ODDS_10_11;

880:     } else if (_numTiers == 12) {

881:       if (_tier == 0) return TIER_ODDS_0_12;

882:       else if (_tier == 1) return TIER_ODDS_1_12;

883:       else if (_tier == 2) return TIER_ODDS_2_12;

884:       else if (_tier == 3) return TIER_ODDS_3_12;

885:       else if (_tier == 4) return TIER_ODDS_4_12;

886:       else if (_tier == 5) return TIER_ODDS_5_12;

887:       else if (_tier == 6) return TIER_ODDS_6_12;

888:       else if (_tier == 7) return TIER_ODDS_7_12;

889:       else if (_tier == 8) return TIER_ODDS_8_12;

890:       else if (_tier == 9) return TIER_ODDS_9_12;

891:       else if (_tier == 10) return TIER_ODDS_10_12;

892:       else if (_tier == 11) return TIER_ODDS_11_12;

893:     } else if (_numTiers == 13) {

894:       if (_tier == 0) return TIER_ODDS_0_13;

895:       else if (_tier == 1) return TIER_ODDS_1_13;

896:       else if (_tier == 2) return TIER_ODDS_2_13;

897:       else if (_tier == 3) return TIER_ODDS_3_13;

898:       else if (_tier == 4) return TIER_ODDS_4_13;

899:       else if (_tier == 5) return TIER_ODDS_5_13;

900:       else if (_tier == 6) return TIER_ODDS_6_13;

901:       else if (_tier == 7) return TIER_ODDS_7_13;

902:       else if (_tier == 8) return TIER_ODDS_8_13;

903:       else if (_tier == 9) return TIER_ODDS_9_13;

904:       else if (_tier == 10) return TIER_ODDS_10_13;

905:       else if (_tier == 11) return TIER_ODDS_11_13;

906:       else if (_tier == 12) return TIER_ODDS_12_13;

907:     } else if (_numTiers == 14) {

908:       if (_tier == 0) return TIER_ODDS_0_14;

909:       else if (_tier == 1) return TIER_ODDS_1_14;

910:       else if (_tier == 2) return TIER_ODDS_2_14;

911:       else if (_tier == 3) return TIER_ODDS_3_14;

912:       else if (_tier == 4) return TIER_ODDS_4_14;

913:       else if (_tier == 5) return TIER_ODDS_5_14;

914:       else if (_tier == 6) return TIER_ODDS_6_14;

915:       else if (_tier == 7) return TIER_ODDS_7_14;

916:       else if (_tier == 8) return TIER_ODDS_8_14;

917:       else if (_tier == 9) return TIER_ODDS_9_14;

918:       else if (_tier == 10) return TIER_ODDS_10_14;

919:       else if (_tier == 11) return TIER_ODDS_11_14;

920:       else if (_tier == 12) return TIER_ODDS_12_14;

921:       else if (_tier == 13) return TIER_ODDS_13_14;

922:     } else if (_numTiers == 15) {

923:       if (_tier == 0) return TIER_ODDS_0_15;

924:       else if (_tier == 1) return TIER_ODDS_1_15;

925:       else if (_tier == 2) return TIER_ODDS_2_15;

926:       else if (_tier == 3) return TIER_ODDS_3_15;

927:       else if (_tier == 4) return TIER_ODDS_4_15;

928:       else if (_tier == 5) return TIER_ODDS_5_15;

929:       else if (_tier == 6) return TIER_ODDS_6_15;

930:       else if (_tier == 7) return TIER_ODDS_7_15;

931:       else if (_tier == 8) return TIER_ODDS_8_15;

932:       else if (_tier == 9) return TIER_ODDS_9_15;

933:       else if (_tier == 10) return TIER_ODDS_10_15;

934:       else if (_tier == 11) return TIER_ODDS_11_15;

935:       else if (_tier == 12) return TIER_ODDS_12_15;

936:       else if (_tier == 13) return TIER_ODDS_13_15;

937:       else if (_tier == 14) return TIER_ODDS_14_15;

```

```solidity
File: libraries/DrawAccumulatorLib.sol

70:     if (_drawId == 0) {

126:     if (ringBufferInfo.cardinality == 0) {

178:     if (ringBufferInfo.cardinality == 0) {

```

```solidity
File: libraries/TierCalculationLib.sol

80:     if (_vaultTwabTotalSupply == 0) {

```

### <a name="NC-2"></a>[NC-2] Lines are too long
Recommended by solidity docs to keep lines to 120 characters or lesser

*Instances (5)*:
```solidity
File: PrizePool.sol

12: import { UD34x4, fromUD60x18 as fromUD60x18toUD34x4, intoUD60x18 as fromUD34x4toUD60x18, toUD34x4 } from "./libraries/UD34x4.sol";

```

```solidity
File: abstract/TieredLiquidityDistributor.sol

10: import { UD34x4, fromUD60x18 as fromUD60x18toUD34x4, intoUD60x18 as fromUD34x4toUD60x18, toUD34x4 } from "../libraries/UD34x4.sol";

```

```solidity
File: libraries/DrawAccumulatorLib.sol

187:             latest observation. This allows us to make assumptions on the value of `lastObservationDrawIdOccurringAtOrBeforeEnd` and removes the need to run a additional

210:             - b = "body". if there are *two* observations between s and e we calculate how much was disbursed. body is (last obs disbursed - first obs disbursed)

```

```solidity
File: libraries/TierCalculationLib.sol

84:             The user-held portion of the total supply is the "winning zone". If the above pseudo-random number falls within the winning zone, the user has won this tier

```

### <a name="NC-3"></a>[NC-3] Event is missing `indexed` fields
Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

*Instances (7)*:
```solidity
File: PrizePool.sol

158:   event DrawClosed(

171:   event WithdrawReserve(address indexed to, uint256 amount);

176:   event IncreaseReserve(address user, uint256 amount);

182:   event ContributePrizeTokens(address indexed vault, uint16 indexed drawId, uint256 amount);

188:   event WithdrawClaimRewards(address indexed to, uint256 amount, uint256 available);

193:   event IncreaseClaimRewards(address indexed to, uint256 amount);

```

```solidity
File: abstract/TieredLiquidityDistributor.sol

41:   event ReserveConsumed(uint256 amount);

```

### <a name="NC-4"></a>[NC-4] Constants should be defined rather than using magic numbers

*Instances (98)*:
```solidity
File: abstract/TieredLiquidityDistributor.sol

84:   SD59x18 internal constant TIER_ODDS_0_3 = SD59x18.wrap(2739726027397260);

85:   SD59x18 internal constant TIER_ODDS_1_3 = SD59x18.wrap(52342392259021369);

87:   SD59x18 internal constant TIER_ODDS_0_4 = SD59x18.wrap(2739726027397260);

88:   SD59x18 internal constant TIER_ODDS_1_4 = SD59x18.wrap(19579642462506911);

89:   SD59x18 internal constant TIER_ODDS_2_4 = SD59x18.wrap(139927275620255366);

91:   SD59x18 internal constant TIER_ODDS_0_5 = SD59x18.wrap(2739726027397260);

92:   SD59x18 internal constant TIER_ODDS_1_5 = SD59x18.wrap(11975133168707466);

93:   SD59x18 internal constant TIER_ODDS_2_5 = SD59x18.wrap(52342392259021369);

94:   SD59x18 internal constant TIER_ODDS_3_5 = SD59x18.wrap(228784597949733865);

96:   SD59x18 internal constant TIER_ODDS_0_6 = SD59x18.wrap(2739726027397260);

97:   SD59x18 internal constant TIER_ODDS_1_6 = SD59x18.wrap(8915910667410451);

98:   SD59x18 internal constant TIER_ODDS_2_6 = SD59x18.wrap(29015114005673871);

99:   SD59x18 internal constant TIER_ODDS_3_6 = SD59x18.wrap(94424100034951094);

100:   SD59x18 internal constant TIER_ODDS_4_6 = SD59x18.wrap(307285046878222004);

102:   SD59x18 internal constant TIER_ODDS_0_7 = SD59x18.wrap(2739726027397260);

103:   SD59x18 internal constant TIER_ODDS_1_7 = SD59x18.wrap(7324128348251604);

104:   SD59x18 internal constant TIER_ODDS_2_7 = SD59x18.wrap(19579642462506911);

105:   SD59x18 internal constant TIER_ODDS_3_7 = SD59x18.wrap(52342392259021369);

106:   SD59x18 internal constant TIER_ODDS_4_7 = SD59x18.wrap(139927275620255366);

107:   SD59x18 internal constant TIER_ODDS_5_7 = SD59x18.wrap(374068544013333694);

109:   SD59x18 internal constant TIER_ODDS_0_8 = SD59x18.wrap(2739726027397260);

110:   SD59x18 internal constant TIER_ODDS_1_8 = SD59x18.wrap(6364275529026907);

111:   SD59x18 internal constant TIER_ODDS_2_8 = SD59x18.wrap(14783961098420314);

112:   SD59x18 internal constant TIER_ODDS_3_8 = SD59x18.wrap(34342558671878193);

113:   SD59x18 internal constant TIER_ODDS_4_8 = SD59x18.wrap(79776409602255901);

114:   SD59x18 internal constant TIER_ODDS_5_8 = SD59x18.wrap(185317453770221528);

115:   SD59x18 internal constant TIER_ODDS_6_8 = SD59x18.wrap(430485137687959592);

117:   SD59x18 internal constant TIER_ODDS_0_9 = SD59x18.wrap(2739726027397260);

118:   SD59x18 internal constant TIER_ODDS_1_9 = SD59x18.wrap(5727877794074876);

119:   SD59x18 internal constant TIER_ODDS_2_9 = SD59x18.wrap(11975133168707466);

120:   SD59x18 internal constant TIER_ODDS_3_9 = SD59x18.wrap(25036116265717087);

121:   SD59x18 internal constant TIER_ODDS_4_9 = SD59x18.wrap(52342392259021369);

123:   SD59x18 internal constant TIER_ODDS_6_9 = SD59x18.wrap(228784597949733865);

124:   SD59x18 internal constant TIER_ODDS_7_9 = SD59x18.wrap(478314329651259628);

126:   SD59x18 internal constant TIER_ODDS_0_10 = SD59x18.wrap(2739726027397260);

127:   SD59x18 internal constant TIER_ODDS_1_10 = SD59x18.wrap(5277233889074595);

129:   SD59x18 internal constant TIER_ODDS_3_10 = SD59x18.wrap(19579642462506911);

130:   SD59x18 internal constant TIER_ODDS_4_10 = SD59x18.wrap(37714118749773489);

131:   SD59x18 internal constant TIER_ODDS_5_10 = SD59x18.wrap(72644572330454226);

132:   SD59x18 internal constant TIER_ODDS_6_10 = SD59x18.wrap(139927275620255366);

133:   SD59x18 internal constant TIER_ODDS_7_10 = SD59x18.wrap(269526570731818992);

134:   SD59x18 internal constant TIER_ODDS_8_10 = SD59x18.wrap(519159484871285957);

136:   SD59x18 internal constant TIER_ODDS_0_11 = SD59x18.wrap(2739726027397260);

137:   SD59x18 internal constant TIER_ODDS_1_11 = SD59x18.wrap(4942383282734483);

138:   SD59x18 internal constant TIER_ODDS_2_11 = SD59x18.wrap(8915910667410451);

139:   SD59x18 internal constant TIER_ODDS_3_11 = SD59x18.wrap(16084034459031666);

140:   SD59x18 internal constant TIER_ODDS_4_11 = SD59x18.wrap(29015114005673871);

141:   SD59x18 internal constant TIER_ODDS_5_11 = SD59x18.wrap(52342392259021369);

142:   SD59x18 internal constant TIER_ODDS_6_11 = SD59x18.wrap(94424100034951094);

143:   SD59x18 internal constant TIER_ODDS_7_11 = SD59x18.wrap(170338234127496669);

144:   SD59x18 internal constant TIER_ODDS_8_11 = SD59x18.wrap(307285046878222004);

145:   SD59x18 internal constant TIER_ODDS_9_11 = SD59x18.wrap(554332974734700411);

147:   SD59x18 internal constant TIER_ODDS_0_12 = SD59x18.wrap(2739726027397260);

148:   SD59x18 internal constant TIER_ODDS_1_12 = SD59x18.wrap(4684280039134314);

149:   SD59x18 internal constant TIER_ODDS_2_12 = SD59x18.wrap(8009005012036743);

150:   SD59x18 internal constant TIER_ODDS_3_12 = SD59x18.wrap(13693494143591795);

151:   SD59x18 internal constant TIER_ODDS_4_12 = SD59x18.wrap(23412618868232833);

152:   SD59x18 internal constant TIER_ODDS_5_12 = SD59x18.wrap(40030011078337707);

153:   SD59x18 internal constant TIER_ODDS_6_12 = SD59x18.wrap(68441800379112721);

154:   SD59x18 internal constant TIER_ODDS_7_12 = SD59x18.wrap(117019204165776974);

155:   SD59x18 internal constant TIER_ODDS_8_12 = SD59x18.wrap(200075013628233217);

156:   SD59x18 internal constant TIER_ODDS_9_12 = SD59x18.wrap(342080698323914461);

157:   SD59x18 internal constant TIER_ODDS_10_12 = SD59x18.wrap(584876652230121477);

159:   SD59x18 internal constant TIER_ODDS_0_13 = SD59x18.wrap(2739726027397260);

160:   SD59x18 internal constant TIER_ODDS_1_13 = SD59x18.wrap(4479520628784180);

161:   SD59x18 internal constant TIER_ODDS_2_13 = SD59x18.wrap(7324128348251604);

162:   SD59x18 internal constant TIER_ODDS_3_13 = SD59x18.wrap(11975133168707466);

163:   SD59x18 internal constant TIER_ODDS_4_13 = SD59x18.wrap(19579642462506911);

165:   SD59x18 internal constant TIER_ODDS_6_13 = SD59x18.wrap(52342392259021369);

166:   SD59x18 internal constant TIER_ODDS_7_13 = SD59x18.wrap(85581121447732876);

167:   SD59x18 internal constant TIER_ODDS_8_13 = SD59x18.wrap(139927275620255366);

168:   SD59x18 internal constant TIER_ODDS_9_13 = SD59x18.wrap(228784597949733866);

169:   SD59x18 internal constant TIER_ODDS_10_13 = SD59x18.wrap(374068544013333694);

170:   SD59x18 internal constant TIER_ODDS_11_13 = SD59x18.wrap(611611432212751966);

172:   SD59x18 internal constant TIER_ODDS_0_14 = SD59x18.wrap(2739726027397260);

173:   SD59x18 internal constant TIER_ODDS_1_14 = SD59x18.wrap(4313269422986724);

174:   SD59x18 internal constant TIER_ODDS_2_14 = SD59x18.wrap(6790566987074365);

176:   SD59x18 internal constant TIER_ODDS_4_14 = SD59x18.wrap(16830807002169641);

177:   SD59x18 internal constant TIER_ODDS_5_14 = SD59x18.wrap(26497468900426949);

178:   SD59x18 internal constant TIER_ODDS_6_14 = SD59x18.wrap(41716113674084931);

179:   SD59x18 internal constant TIER_ODDS_7_14 = SD59x18.wrap(65675485708038160);

181:   SD59x18 internal constant TIER_ODDS_9_14 = SD59x18.wrap(162780431564813557);

183:   SD59x18 internal constant TIER_ODDS_11_14 = SD59x18.wrap(403460570024895441);

184:   SD59x18 internal constant TIER_ODDS_12_14 = SD59x18.wrap(635185461125249183);

186:   SD59x18 internal constant TIER_ODDS_0_15 = SD59x18.wrap(2739726027397260);

187:   SD59x18 internal constant TIER_ODDS_1_15 = SD59x18.wrap(4175688124417637);

188:   SD59x18 internal constant TIER_ODDS_2_15 = SD59x18.wrap(6364275529026907);

189:   SD59x18 internal constant TIER_ODDS_3_15 = SD59x18.wrap(9699958857683993);

190:   SD59x18 internal constant TIER_ODDS_4_15 = SD59x18.wrap(14783961098420314);

191:   SD59x18 internal constant TIER_ODDS_5_15 = SD59x18.wrap(22532621938542004);

192:   SD59x18 internal constant TIER_ODDS_6_15 = SD59x18.wrap(34342558671878193);

193:   SD59x18 internal constant TIER_ODDS_7_15 = SD59x18.wrap(52342392259021369);

194:   SD59x18 internal constant TIER_ODDS_8_15 = SD59x18.wrap(79776409602255901);

195:   SD59x18 internal constant TIER_ODDS_9_15 = SD59x18.wrap(121589313257458259);

196:   SD59x18 internal constant TIER_ODDS_10_15 = SD59x18.wrap(185317453770221528);

197:   SD59x18 internal constant TIER_ODDS_11_15 = SD59x18.wrap(282447180198804430);

198:   SD59x18 internal constant TIER_ODDS_12_15 = SD59x18.wrap(430485137687959592);

199:   SD59x18 internal constant TIER_ODDS_13_15 = SD59x18.wrap(656113662171395111);

```

### <a name="NC-5"></a>[NC-5] Variable names don't follow the Solidity style guide
Constant variable should be all caps  each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

*Instances (2)*:
```solidity
File: libraries/UD34x4.sol

14: uint128 constant uMAX_UD34x4 = 340282366920938463463374607431768211455;

16: uint128 constant uUNIT = 1e4;

```

