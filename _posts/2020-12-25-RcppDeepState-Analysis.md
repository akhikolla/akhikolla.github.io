---
title: RcppDeepState Analysis
author: Akhila Chowdary Kolla
categories: [RcppDeepState Results,Packages]
tags: [R,Valgrind,PackagesIssues,deepstate,FuzzerIssues]
math: true
---

Over the past few months, I have been working on testing the Rcpp packages using RcppDeepState in the cluster. When we fuzz test each Rcpp function-specific testharness I have found issues in 134 packages. 

Refer the [web page](https://akhikolla.github.io./packages-folders/root.html) to check if your Rcpp package has any subtle bugs. This web page lists the packages with Issues in exported functions and unexported functions.

> total_counts()
$ADtools
[1] 1

$AGread
[1] 7

$ALassoSurvIC
[1] 3

$AdaptGauss
[1] 1

$BAREB
[1] 1

$BCT
[1] 2

$BHSBVAR
[1] 6

$BIFIEsurvey
[1] 23

$BNPMIXcluster
[1] 1

$BNPmix
[1] 1

$BNSL
[1] 20

$BSL
[1] 2

$BTM
[1] 1

$BWStest
[1] 3

$BacArena
[1] 2

$BalancedSampling
[1] 17

$Barycenter
[1] 1

$BayesESS
[1] 2

$BayesFactor
[1] 9

$BayesMallows
[1] 2

$BayesSUR
[1] 8

$BayesianTools
[1] 1

$Benchmarking
[1] 10

$Bestie
[1] 1

$BiDAG
[1] 6

$BigVAR
[1] 2

$Bioi
[1] 1

$BondValuation
[1] 14

$BoostMLR
[1] 37

$CARBayes
[1] 23

$CARBayesST
[1] 71

$CDM
[1] 18

$CENFA
[1] 3

$CGGP
[1] 4

$CKLRT
[1] 12

$CMF
[1] 5

$CPAT
[1] 2

$CVXR
[1] 1

$CatReg
[1] 1

$CautiousLearning
[1] 2

$ChangePointTaylor
[1] 4

$CircSpaceTime
[1] 1

$CircularDDM
[1] 2

$ClinicalTrialSummary
[1] 1

$ClusVis
[1] 2

$ClustMMDD
[1] 5

$ClusterR
[1] 1

$ClusterStability
[1] 2

$CovTools
[1] 1

$DCEM
[1] 2

$DEploid
[1] 1

$DPWeibull
[1] 6

$DStree
[1] 4

$DTDA.cif
[1] 1

$DataVisualizations
[1] 2

$DatabionicSwarm
[1] 3

$DescTools
[1] 6

$DriftBurstHypothesis
[1] 1

$DtD
[1] 1

$EBMAforecast
[1] 8

$EFAtools
[1] 2

$EMMIXgene
[1] 2

$EMbC
[1] 2

$ETAS
[1] 7

$EWGoF
[1] 8

$EloChoice
[1] 2

$EloRating
[1] 1

$EstMix
[1] 2

$ExactMultinom
[1] 1

$FDRSeg
[1] 6

$FENmlm
[1] 13

$FKSUM
[1] 10

$FLSSS
[1] 5

$FRK
[1] 1

$FSSF
[1] 1

$FarmSelect
[1] 2

$FarmTest
[1] 1

$FastGP
[1] 3

$FastGaSP
[1] 2

$FeatureHashing
[1] 6

$FlyingR
[1] 4

$GA
[1] 6

$GAS
[1] 1

$GBJ
[1] 1

$GDINA
[1] 3

$GEEaSPU
[1] 1

$GGClassification
[1] 1

$GGIR
[1] 2

$GGMncv
[1] 1

$GLMaSPU
[1] 1

$GMKMcharlie
[1] 4

$GPGame
[1] 2

$GPM
[1] 1

$GPareto
[1] 6

$GPvecchia
[1] 3

$GUILDS
[1] 1

$GWmodel
[1] 1

$GauPro
[1] 1

$GeDS
[1] 8

$GenEst
[1] 2

$GeneralizedUmatrix
[1] 2

$GenomicMating
[1] 1

$GillespieSSA2
[1] 3

$Gmedian
[1] 1

$Gmisc
[1] 1

$GpGp
[1] 1

$GreedyExperimentalDesign
[1] 2

$HARModel
[1] 1

$HBV.IANIGLA
[1] 8

$HDLSSkST
[1] 3

$HHG
[1] 4

$HRM
[1] 3

$HardyWeinberg
[1] 1

$HistDAWass
[1] 11

$ICcalib
[1] 4

$ICtest
[1] 1

$IFC
[1] 8

$IHSEP
[1] 1

$IOHanalyzer
[1] 1

$ImpactEffectsize
[1] 3

$IncDTW
[1] 1

$IntegratedMRF
[1] 2

$IntervalSurgeon
[1] 2

$IsingSampler
[1] 5

$JuliaCall
[1] 1

$KONPsurv
[1] 4

$KSgeneral
[1] 1

$KoulMde
[1] 1

$LAM
[1] 2

$LANDD
[1] 1

$LBSPR
[1] 2

$LOPART
[1] 1

$LassoBacktracking
[1] 6

$LassoNet
[1] 1

$Lmoments
[1] 1

$LongMemoryTS
[1] 1

$Luminescence
[1] 1

$MAPITR
[1] 1

$MAVE
[1] 1

$MCMCprecision
[1] 4

$MESS
[1] 8

$MGMM
[1] 1

$MM4LMM
[1] 1

$MMVBVS
[1] 1

$MPBoost
[1] 1

$MatrixCorrelation
[1] 1

$MiSPU
[1] 1

$MixMatrix
[1] 1

$ModelMetrics
[1] 24

$MomTrunc
[1] 2

$MuChPoint
[1] 1

$MultivariateRandomForest
[1] 2

$NAM
[1] 36

$NCutYX
[1] 1

$OTclust
[1] 1

$OpenImageR
[1] 1

$Opt5PL
[1] 26

$OwenQ
[1] 15

$PAutilities
[1] 2

$PLMIX
[1] 11

$POMaSPU
[1] 2

$PP
[1] 8

$PPRL
[1] 4

$PPforest
[1] 4

$PPtreeViz
[1] 2

$PWD
[1] 1

$PanelCount
[1] 4

$PanelMatch
[1] 7

$PenCoxFrail
[1] 1

$Phase12Compare
[1] 2

$PoissonBinomial
[1] 12

$PolynomF
[1] 2

$ProFound
[1] 5

$ProbReco
[1] 1

$ProjectionBasedClustering
[1] 3

$QF
[1] 2

$QRM
[1] 2

$QTL.gCIMapping
[1] 1

$QTL.gCIMapping.GUI
[1] 1

$Qtools
[1] 6

$QuantTools
[1] 4

$Quartet
[1] 15

$RAINBOWR
[1] 5

$REddyProc
[1] 1

$RGeode
[1] 4

$RI2by2
[1] 1

$RJafroc
[1] 30

$RMPSH
[1] 3

$RNOmni
[1] 1

$ROI.plugin.qpoases
[1] 3

$ROptSpace
[1] 1

$RPEGLMEN
[1] 1

$RSSL
[1] 1

$RScelestial
[1] 1

$RStoolbox
[1] 3

$RTCC
[1] 1

$RTransferEntropy
[1] 1

$RZigZag
[1] 1

$RaceID
[1] 4

$RcppAPT
[1] 1

$RcppCWB
[1] 1

$RcppDist
[1] 25

$RcppDynProg
[1] 17

$RcppEigen
[1] 1

$RcppExamples
[1] 1

$RcppHungarian
[1] 1

$RcppNumerical
[1] 1

$RcppRoll
[1] 1

$RcppStreams
[1] 1

$RcppZiggurat
[1] 16

$Rdimtools
[1] 1

$ReIns
[1] 2

$RestRserve
[1] 3

$Rfssa
[1] 1

$Riemann
[1] 3

$Rip46
[1] 8

$Rirt
[1] 5

$Rlda
[1] 3

$Rlibeemd
[1] 2

$RobGARCHBoot
[1] 6

$RobustCalibration
[1] 1

$Rphylopars
[1] 1

$Rpvt
[1] 41

$Rrelperm
[1] 8

$Rtsne
[1] 1

$Rvoterdistance
[1] 6

$SHT
[1] 1

$SelvarMix
[1] 1

$SeqNet
[1] 6

$ShrinkCovMat
[1] 1

$SpaDES.tools
[1] 4

$SpatPCA
[1] 1

$SpatialEpi
[1] 9

$Spbsampling
[1] 1

$SpecsVerification
[1] 3

$StMoSim
[1] 1

$StepSignalMargiLike
[1] 4

$SuperGauss
[1] 5

$SuperpixelImageSegmentation
[1] 1

$T4cluster
[1] 1

$TAG
[1] 1

$TAM
[1] 28

$TLMoments
[1] 9

$TSrepr
[1] 26

$TauStar
[1] 3

$Temporal
[1] 1

$TestDesign
[1] 1

$TransPhylo
[1] 8

$TruncatedNormal
[1] 1

$Umatrix
[1] 6

$VARtests
[1] 1

$VNM
[1] 28

$VUROCS
[1] 1

$VeryLargeIntegers
[1] 11

$WebGestaltR
[1] 1

$WinRatio
[1] 5

$YPInterimTesting
[1] 1

$ZVCV
[1] 1

$abcADM
[1] 5

$abcrf
[1] 3

$abtest
[1] 6

$acc
[1] 4

$accelerometry
[1] 17

$activegp
[1] 18

$adheRenceRX
[1] 2

$alphabetr
[1] 3

$ambient
[1] 28

$amt
[1] 16

$anytime
[1] 5

$apcf
[1] 1

$aphid
[1] 8

$apollo
[1] 1

$aricode
[1] 3

$atakrig
[1] 1

$augSIMEX
[1] 7

$autothresholdr
[1] 21

$bWGR
[1] 32

$backbone
[1] 2

$bartBMA
[1] 56

$bayesAB
[1] 2

$bayescopulareg
[1] 1

$bayesm
[1] 1

$bayesmove
[1] 2

$bbl
[1] 1

$bcROCsurface
[1] 4

$beyondWhittle
[1] 10

$bggum
[1] 5

$biClassify
[1] 3

$bigReg
[1] 1

$bigtime
[1] 2

$binnednp
[1] 28

$binsegRcpp
[1] 1

$bisque
[1] 1

$biwavelet
[1] 4

$blatent
[1] 2

$blindrecalc
[1] 1

$blockForest
[1] 1

$blorr
[1] 1

$bmotif
[1] 96

$bnclassify
[1] 5

$bnnSurvival
[1] 1

$borrowr
[1] 2

$bpnreg
[1] 1

$bravo
[1] 2

$breakfast
[1] 1

$bsearchtools
[1] 8

$bsplinePsd
[1] 15

$bvarsv
[1] 1

$bvartools
[1] 1

$carSurv
[1] 4

$castor
[1] 1

$catsim
[1] 8

$cbinom
[1] 1

$cenROC
[1] 2

$cgAUC
[1] 4

$changepointsHD
[1] 1

$chickn
[1] 4

$clogitL1
[1] 2

$clogitboost
[1] 4

$clusrank
[1] 9

$clustermq
[1] 1

$cmenet
[1] 1

$cna
[1] 8

$coda.base
[1] 1

$coga
[1] 24

$colorednoise
[1] 5

$colourvalues
[1] 6

$comat
[1] 2

$compas
[1] 2

$conquer
[1] 1

$conquestr
[1] 2

$cord
[1] 1

$corpustools
[1] 1

$corrcoverage
[1] 1

$cort
[1] 10

$covTestR
[1] 1

$covglasso
[1] 1

$cppRouting
[1] 6

$cpr
[1] 1

$cpss
[1] 2

$crandep
[1] 2

$cstab
[1] 3

$ctmcd
[1] 2

$cutpointr
[1] 6

$cycleRtools
[1] 9

$cytometree
[1] 2

$dHSIC
[1] 4

$dann
[1] 1

$datafsm
[1] 1

$dbmss
[1] 1

$dbscan
[1] 13

$dcov
[1] 1

$dcurver
[1] 3

$ddsPLS
[1] 3

$decido
[1] 1

$densEstBayes
[1] 1

$detectRUNS
[1] 4

$detrendr
[1] 25

$dfConn
[1] 2

$dfphase1
[1] 4

$dgumbel
[1] 1

$diceR
[1] 2

$diffrprojects
[1] 3

$dipsaus
[1] 1

$disclapmix
[1] 8

$distr6
[1] 3

$divDyn
[1] 3

$diveRsity
[1] 7

$diversityForest
[1] 1

$dng
[1] 14

$dpseg
[1] 2

$dqrng
[1] 4

$dscore
[1] 3

$dsmisc
[1] 1

$durmod
[1] 1

$dvmisc
[1] 18

$dynamichazard
[1] 1

$dynutils
[1] 1

$eBsc
[1] 3

$eDMA
[1] 1

$earthtide
[1] 4

$easyVerification
[1] 3

$ebmstate
[1] 2

$eive
[1] 1

$elmNNRcpp
[1] 1

$elo
[1] 4

$energy
[1] 10

$equateMultiple
[1] 5

$ergmito
[1] 1

$errum
[1] 1

$eseis
[1] 2

$esreg
[1] 7

$essHist
[1] 1

$evgam
[1] 1

$evolqg
[1] 1

$exceedProb
[1] 1

$expperm
[1] 4

$factorcpt
[1] 5

$fad
[1] 2

$fastAdaboost
[1] 1

$fastLink
[1] 1

$fasteraster
[1] 1

$fastmit
[1] 1

$fastshap
[1] 1

$fbroc
[1] 17

$fclust
[1] 1

$fctbases
[1] 2

$fdadensity
[1] 2

$fdapace
[1] 5

$ffstream
[1] 20

$finity
[1] 1

$fixest
[1] 33

$flam
[1] 7

$flexpolyline
[1] 1

$flock
[1] 1

$flying
[1] 2

$foreSIGHT
[1] 3

$forecastSNSTS
[1] 3

$forestControl
[1] 2

$fplot
[1] 2

$fractional
[1] 1

$fracture
[1] 2

$frailtyEM
[1] 2

$frailtySurv
[1] 23

$fromo
[1] 5

$futureheatwaves
[1] 1

$gRbase
[1] 3

$gapfill
[1] 1

$gbeta
[1] 1

$gdalcubes
[1] 1

$gdtools
[1] 1

$gear
[1] 4

$geometr
[1] 1

$ggforce
[1] 4

$ggiraph
[1] 3

$ggraph
[1] 5

$ggrepel
[1] 7

$gjam
[1] 3

$glca
[1] 3

$glmBfp
[1] 1

$glow
[1] 1

$goffda
[1] 1

$googlePolylines
[1] 1

$grainscape
[1] 1

$graphicalVAR
[1] 3

$graphlayouts
[1] 10

$grattan
[1] 13

$gretel
[1] 2

$gridtext
[1] 2

$groupedSurv
[1] 3

$grpCox
[1] 2

$grpSLOPE
[1] 1

$gsEasy
[1] 2

$gscounts
[1] 1

$gsynth
[1] 1

$gtfs2gps
[1] 1

$hal9001
[1] 1

$hans
[1] 1

$hdbinseg
[1] 8

$heatwaveR
[1] 1

$hermiter
[1] 8

$hesim
[1] 1

$hetGP
[1] 42

$heuristicsmineR
[1] 4

$hierarchicalSets
[1] 3

$highfrequency
[1] 9

$hilbertSimilarity
[1] 1

$hmlasso
[1] 3

$hpa
[1] 1

$httpuv
[1] 4

$huge
[1] 1

$humaniformat
[1] 4

$humanleague
[1] 3

$hutilscpp
[1] 34

$hypervolume
[1] 1

$iBATCGH
[1] 1

$iCellR
[1] 1

$iRF
[1] 1

$ibdsim2
[1] 1

$ibmcraftr
[1] 1

$ibs
[1] 3

$icRSF
[1] 15

$icenReg
[1] 2

$icensmis
[1] 29

$icosa
[1] 41

$idefix
[1] 1

$iilasso
[1] 3

$image.ContourDetector
[1] 1

$image.LineSegmentDetector
[1] 1

$image.Otsu
[1] 1

$image.libfacedetection
[1] 1

$imager
[1] 43

$imagerExtra
[1] 19

$imagine
[1] 5

$immer
[1] 35

$immunarch
[1] 2

$imp4p
[1] 5

$infinitefactor
[1] 1

$inspectdf
[1] 5

$intcensROC
[1] 3

$interp
[1] 5

$ipaddress
[1] 3

$ipft
[1] 5

$iprior
[1] 1

$irt
[1] 2

$ivsacim
[1] 1

$ivtools
[1] 1

$jaccard
[1] 2

$jmotif
[1] 20

$junctions
[1] 2

$kcpRS
[1] 1

$kmer
[1] 1

$ksNN
[1] 1

$lefko3
[1] 1

$lsbclust
[1] 5

$lutz
[1] 1

$metacart
[1] 2

$metadynminer3d
[1] 24

$meteoland
[1] 47

$mixR
[1] 40

$mlr3proba
[1] 4

$mmsample
[1] 1

$mnlfa
[1] 2

$mobsim
[1] 2

$mosum
[1] 4

$mousetrap
[1] 30

$multbxxc
[1] 1

$multdyn
[1] 1

$multicool
[1] 3

$multinets
[1] 1

$multinomineq
[1] 3

$mwaved
[1] 6

$nandb
[1] 4

$ndjson
[1] 1

$nmixgof
[1] 2

$oce
[1] 24

$offlineChange
[1] 1

$olctools
[1] 7

$openxlsx
[1] 11

$ordinalClust
[1] 5

$padr
[1] 4

$particles
[1] 5

$parzer
[1] 5

$pbv
[1] 4

$pcIRT
[1] 1

$perccal
[1] 3

$ph2bye
[1] 1

$phangorn
[1] 3

$phenofit
[1] 9

$phylobase
[1] 14

$pinbasic
[1] 2

$propr
[1] 11

$pryr
[1] 2

$psd
[1] 4

$psrwe
[1] 2

$psychonetrics
[1] 1

$ptsuite
[1] 10

$puniform
[1] 2

$quantregRanger
[1] 5

$qwraps2
[1] 2

$rENA
[1] 1

$rankdist
[1] 10

$rbscCI
[1] 2

$reda
[1] 1

$redist
[1] 12

$refinr
[1] 2

$registr
[1] 1

$relSim
[1] 4

$rem
[1] 1

$remote
[1] 7

$repolr
[1] 2

$resemble
[1] 2

$rhoR
[1] 5

$robFitConGraph
[1] 2

$robmixglm
[1] 4

$robustSingleCell
[1] 1

$rrecsys
[1] 8

$rres
[1] 1

$satellite
[1] 3

$segmenTier
[1] 4

$segmentr
[1] 1

$seismicRoll
[1] 6

$selectiveInference
[1] 9

$sensitivity
[1] 13

$sentometrics
[1] 1

$seqest
[1] 1

$serrsBayes
[1] 6

$sf
[1] 15

$signalHsmm
[1] 1

$signnet
[1] 3

$simPop
[1] 7

$simcdm
[1] 1

$simmer
[1] 4

$simstudy
[1] 3

$simts
[1] 4

$sirt
[1] 26

$splithalf
[1] 1

$stocks
[1] 8

$striprtf
[1] 1

$supc
[1] 2

$superml
[1] 4

$sureLDA
[1] 1

$surveybootstrap
[1] 1

$surveysd
[1] 5

$survivalmodels
[1] 1

$svars
[1] 1

$swephR
[1] 41

$swmmr
[1] 1

$symengine
[1] 1

$systemicrisk
[1] 1

$tagcloud
[1] 3

$tbart
[1] 6

$teachingApps
[1] 23

$telefit
[1] 1

$tergmLite
[1] 1

$tidyxl
[1] 2

$timma
[1] 6

$tmt
[1] 3

$twosamples
[1] 7

$udpipe
[1] 1

$umap
[1] 1

$understandBPMN
[1] 1

$units
[1] 10

$unsystation
[1] 6

$updog
[1] 40

$urltools
[1] 14

$uwot
[1] 2

$vapour
[1] 12

$varbvs
[1] 1

$vcfR
[1] 1

$vegclust
[1] 3

$vennplot
[1] 6

$vlad
[1] 9

$volesti
[1] 1

$vsgoftest
[1] 2

$waspr
[1] 2

$wbsts
[1] 4

$webreadr
[1] 2

$weibulltools
[1] 2

$wellknown
[1] 6

$wicket
[1] 5

$windfarmGA
[1] 4

$wk
[1] 4

$wkutils
[1] 10

$wordspace
[1] 9

$wrswoR
[1] 4

$wv
[1] 2

$xdcclarge
[1] 4

$xyz
[1] 10

$yuima
[1] 10

$zcurve
[1] 11

$ziphsmm
[1] 1

$ztpln
[1] 8

Total functions evaluated :
3453
Total packages evaluated : 
653
empty testfiles packages : 
666
empty testfiles packages : 
list("ABCoptim", "ADMM", "ADMMnet", "ADMMsigma", "ANN2", "APML0", "ARCokrig", "ASPBay", "AhoCorasickTrie", "AlphaSimR", "BAMBI", "BCEE", "BCSub", "BClustLonG", "BET", "BGGM", "BGVAR", "BLPestimatoR", "BLSM", "BMTME", "BOSSreg", "BTLLasso", "BVSNLP", "BayesLN", "BayesMFSurv", "BayesMRA", "BayesProject", "BayesReversePLLH", "BayesSPsurv", "Bayesrel", "BeSS", "BeviMed", "BiProbitPartial", "BigQuic", "BinSegBstrap", "BinaryDosage", "Biocomb", "BoltzMM", "Buddle", "BuyseTest", "CASMAP", "CB2", "CFC", 
    "CLVTools", "Ckmeans.1d.dp", "ClustVarLV", "CoMiRe", "Compack", "Countr", "Cyclops", "DCPO", "DDRTree", "DatAssim", "DataGraph", "DataViz", "DiffNet", "DiscreteFDR", "DstarM", "DynComm", "DysPIA", "EAinference", "ECctmc", "EMVS", "Eagle", "EditImputeCont", "EigenR", "EpiNow2", "EstHer", "FDRreg", "FDX", "FIT", "FMCCSD", "FSInteract", "FSelectorRcpp", "FastBandChol", "FastSF", "FiRE", "FisHiCal", "FisPro", "FlexReg", "FunChisq", "GADAG", "GCSM", "GLMcat", "GMCM", "GPCMlasso", "GUTS", "GiniDistance", 
    "GofKmt", "GreedyEPL", "GreedySBTM", "GridOnClusters", "GxEScanR", "HACSim", "HDclust", "HMB", "HMMmlselect", "HSAR", "HTLR", "HUM", "HypergeoMat", "ICAOD", "ICRanks", "IMAGE", "IOHexperimenter", "ISOpureR", "IceCast", "Immigrate", "IrishDirectorates", "IsoSpecR", "JMI", "JMbayes", "JMcmprsk", "JOUSBoost", "JSM", "JacobiEigen", "JumpTest", "KernelKnn", "L0Learn", "L1mstate", "LDheatmap", "LambertW", "Langevin", "LassoGEE", "LeMaRns", "LocalControl", "LowWAFOMNX", "LowWAFOMSobol", "MABOUST", "MADPop", 
    "MAT", "MCPModPack", "MEGENA", "MGDrivE", "MHMM", "MIDASwrappeR", "MLModelSelection", "MPTinR", "MRS", "MSGARCH", "MTLR", "MatchIt", "MatrixLDA", "MediaK", "Mega2R", "MetaheuristicFPA", "MrSGUIDE", "MultiBD", "MultiFit", "N2R", "NHMM", "NPBayesImputeCat", "NetRep", "NetworkInference", "NlinTS", "OncoBayes2", "OneArmPhaseTwoStudy", "OptCirClust", "Orcs", "PAC", "PAFit", "PAsso", "PCMRS", "PINSPlus", "PLNmodels", "PO.EN", "PPMR", "PQLseq", "PRDA", "PRIMME", "PUlasso", "PandemicLP", "Peptides", 
    "Phase123", "PhenotypeSimulator", "PhylogeneticEM", "PieceExpIntensity", "Pijavski", "ProcData", "RBesT", "RCSF", "RClickhouse", "REndo", "RInside", "RJcluster", "RKHSMetaMod", "RLRsim", "RLumCarlo", "RLumModel", "RMKL", "RMSNumpress", "RMariaDB", "RMixtCompIO", "RNewsflow", "RPostgres", "RPresto", "RPtests", "RQuantLib", "RRI", "RSQLite", "RVowpalWabbit", "RWDataPlyr", "Radviz", "Rblpapi", "RcppAlgos", "RcppAnnoy", "RcppArmadillo", "RcppBigIntAlgos", "RcppCCTZ", "RcppCNPy", "RcppDE", "RcppEigenAD", 
    "RcppEnsmallen", "RcppFaddeeva", "RcppGSL", "RcppGetconf", "RcppGreedySetCover", "RcppHNSW", "RcppMLPACK", "RcppMeCab", "RcppMsgPack", "RcppNLoptExample", "RcppQuantuccia", "RcppSMC", "RcppSimdJson", "RcppSpdlog", "RcppTOML", "RcppUUID", "RcppXsimd", "Rdca", "Rdtq", "RecAssoRules", "ReorderCluster", "RiemBase", "Rlabkey", "Rlinsolve", "RmecabKo", "Rnmr1D", "RobKF", "RobustGaSP", "Ropj", "RoughSets", "Rquefts", "RstoxData", "Rwofost", "Ryacas", "Ryacas0", "SAGMM", "SAMM", "SAR", "SBmedian", "SCORNET", 
    "SCPME", "SDMtune", "SEERaBomb", "SELF", "SFS", "SILGGM", "SITH", "SLOPE", "SMITIDvisu", "SMMA", "SMUT", "SPARSEMODr", "STARTdesign", "SeqDetect", "SeqKat", "SequenceSpikeSlab", "Seurat", "Signac", "SimBIID", "SimSurvNMarker", "SmartSVA", "SobolSequence", "SpaCCr", "SpaTimeClus", "SparseFactorAnalysis", "SparseLPM", "SpatialBSS", "SpatialKDE", "SplitReg", "SubTite", "SuperRanker", "SurvBoost", "T4transport", "TBRDist", "TDAstats", "TESS", "TSDFGS", "TestCor", "TexExamRandomizer", "TreeBUGS", 
    "TreeLS", "UComp", "UniDOE", "UniIsoRegression", "UtilityFrailtyPH12", "V8", "WRI", "WaveSampling", "YPBP", "acrt", "adpss", "afCEC", "agtboost", "alakazam", "alpaca", "ampir", "anMC", "anomaly", "ape", "armspp", "arrApply", "ashr", "autoFRK", "avar", "bama", "batman", "bayesDP", "bayesGAM", "bayesbr", "bcf", "bcp", "bcpa", "beam", "belg", "bellreg", "benchr", "bife", "bigMap", "bigSurvSGD", "biglasso", "bigmemory", "bigreadr", "bigrquery", "bigsnpr", "bigsparser", "bigstatsr", "bigutilsr", "binGroup2", 
    "binaryGP", "bindrcpp", "bio3d", "bioacoustics", "blackbox", "bmlm", "bmrm", "bootUR", "bpcs", "bpgmm", "breathteststan", "cIRT", "cattonum", "causaloptim", "cblasr", "cbq", "cccp", "ccdrAlgorithm", "cctools", "ced", "cellWise", "cglm", "changepoint.mv", "chillR", "chopthin", "chunkR", "cld2", "cld3", "clevr", "clifford", "cnaOpt", "cnum", "coala", "collUtils", "combiter", "comparator", "compboost", "comperank", "contoureR", "copCAR", "corels", "coxmeg", "coxrt", "cqrReg", "crawl", "crfsuite", 
    "ctgt", "cubature", "curstatCI", "curveDepth", "cusum", "datasailr", "dbnR", "deepboost", "demu", "depcoeff", "dexter", "dexterMST", "diffman", "diffusr", "dina", "discretecdAlgorithm", "disk.frame", "dmbc", "doc2vec", "dodgr", "dosearch", "downscaledl", "dracor", "drf", "dslice", "dynmix", "dynsbm", "eddington", "edina", "eimpute", "elfDistr", "empichar", "emstreeR", "epinetr", "ess", "estimatr", "estudy2", "exactextractr", "exdex", "exif", "expSBM", "extraDistr", "exuber", "fDMA", "fRLR", "fabMix", 
    "facilitation", "fastJT", "fastTextR", "fastcmh", "fasterElasticNet", "fasterize", "fastglm", "fastlogranktest", "fastpos", "fddm", "feather", "filling", "fingerPro", "fishflux", "flamingos", "flars", "flexsurv", "fmf", "fourPNO", "fourierin", "fpeek", "freealg", "fst", "fstcore", "fuser", "fwsim", "gRain", "gRim", "gamreg", "gasper", "gastempt", "gbp", "gdpc", "gee4", "gen3sis", "genepop", "genie", "genieclust", "genio", "genodds", "geoFKF", "geodiv", "geogrid", "geojsonR", "geojsonsf", "geometries", 
    "geometry", "geoops", "gfiExtremes", "gfilinreg", "gfilmm", "gfpop", "ggdmc", "ggip", "ggwordcloud", "gif", "gkmSVM", "glamlasso", "glcm", "glm.deploy", "glmaag", "glmdisc", "glmmsr", "goldi", "graphkernels", "graphql", "gren", "greybox", "grf", "grove", "gsisdecoder", "gtfsrouter", "gwfa", "hadron", "haven", "hawkes", "hdbm", "hdme", "hdpGLM", "higlasso", "hipread", "hit", "hkevp", "hommel", "hopit", "hsrecombi", "htdp", "htmltidy", "hts", "hunspell", "hyper2", "iccbeta", "icr", "ideq", "image.CannyEdges", 
    "image.CornerDetectionF9", "image.CornerDetectionHarris", "image.binarization", "image.dlib", "image.textlinedetector", "imbalance", "impactflu", "imptree", "imputeMulti", "incgraph", "independence", "inplace", "interep", "interleave", "intervalaverage", "intkrige", "intsurv", "isotree", "ivx", "jackalope", "jmvconnect", "jti", "kde1d", "kdtools", "latentgraph", "lazygreedy", "leidenAlg", "lslx", "mmapcharr", "mnis", "mniw", "modeLLtest", "molic", "mombf", "mrMLM", "multinet", "multinma", "multivar", 
    "multivariance", "mvabund", "mvcluster", "mvp", "mvrsquared", "myTAI", "nVennR", "odbc", "oppr", "optiSel", "optimization", "optmatch", "opusminer", "ordinalForest", "outliertree", "proxyC", "pseudorank", "psqn", "pts2polys", "pvar", "qmix", "r2sundials", "rCRM", "rDotNet", "rEDM", "rIsing", "rasterly", "regmed", "regnet", "reticulate", "revdbayes", "ripserr", "rocTree", "roll", "rollRegres", "rootWishart", "roptim", "rotations", "saturnin", "sbfc", "sbm", "sbmSDP", "sbo", "sboost", "semver", 
    "sentencepiece", "seqHMM", "set6", "simcross", "simplextree", "sitmo", "skpr", "splines2", "splmm", "spnn", "sport", "spray", "sprintr", "strider", "stringfish", "sundialr", "surbayes", "surveillance", "svgViewR", "svglite", "symmetry", "synchronicity", "synthACS", "targeted", "tcR", "tensorBSS", "tokenbrowser", "tokenizers", "tokenizers.bpe", "torch", "touch", "tracerer", "tree.interpreter", "uFTIR", "ulid", "unine", "valr", "valuer", "varband", "vcpen", "vdiffr", "vinereg", "viscomplexr", "vita", 
    "wCorr", "walker", "wbsd", "wdm", "websocket", "womblR", "word2vec", "wordcloud", "xrnet", "xslt", "xtensor", "yakmoR")
No Rcpp exports file for pkg - 
list("ACEt", "AdaptiveSparsity", "AlphaPart", "Amelia", "AnaCoDa", "BART", "BaBooN", "BayesComm", "BivRec", "CNVRG", "CNull", "COMPoissonReg", "CVR", "CamelUp", "CausalQueries", "ChannelAttribution", "ConConPiWiFun", "CoxPlus", "DLMtool", "DMCfun", "DNAtools", "DPP", "DeCAFS", "DeLorean", "DepthProc", "DetMCD", "DetR", "DiffusionRgqd", "DiffusionRimp", "DiffusionRjgqd", "DrImpute", "FBFsearch", "FRESA.CAD", "FastHCS", "FastPCS", "FastRCS", "FaultTree", "GCPM", "GENLIB", "GMMAT", "GPvam", "GSE", "GenomicTools", 
    "GiRaF", "HDPenReg", "HLMdiag", "HMMEsolver", "IMaGES", "KernSmoothIRT", "LARisk", "LaF", "MAINT.Data", "MTS", "MVB", "ManifoldOptim", "MetaStan", "MixAll", "Morpho", "NLMR", "NPflow", "NestedCategBayesImpute", "NetMix", "NetworkDistance", "Numero", "OjaNP", "OpenMx", "PCMBaseCpp", "POUMM", "PRIMAL", "PReMiuM", "PedCNV", "PerMallows", "PoweR", "RNifti", "RNiftyReg", "RProtoBuf", "RSNNS", "RSNPset", "RSpectra", "Rankcluster", "Ravages", "Rborist", "RcppBDT", "RcppClassic", "RcppClassicExamples", 
    "RcppDL", "RcppHMM", "RcppRedis", "RcppTN", "RcppXts", "RealVAMS", "Rfast", "Rfast2", "Rlgt", "Rmalschains", "Rmixmod", "Rvcg", "SAM", "SBSA", "SCCI", "SNPknock", "SSLR", "SSOSVM", "STARTS", "Scalelink", "SimJoint", "SimReg", "SimilaR", "SocialNetworks", "SpatialTools", "StereoMorph", "SurrogateRegression", "TAQMNGR", "TDA", "TFMPvalue", "TreeDist", "TreeTools", "VIM", "VarSelLCM", "WGCNA", "XBRL", "acebayes", "apcluster", "baggr", "basad", "bayes4psy", "bayesImageS", "bayesdfa", "beanz", "bfp", 
    "biganalytics", "bigtabulate", "blavaan", "blockcluster", "blockmodels", "bmgarch", "ccaPP", "cladoRcpp", "clere", "climdex.pcic", "clusternor", "clusteval", "coneproj", "ddalpha", "dfcomb", "dfmta", "dfpk", "dils", "disaggR", "diversitree", "divest", "dlib", "drgee", "dtwclust", "ecp", "eggCounts", "emIRT", "esaddle", "etm", "fable", "factorstochvol", "fastGHQuad", "fdaMixed", "fdasrvf", "flan", "forecast", "fssemR", "fugeR", "fusedest", "gMWT", "gaselect", "gaston", "gcKrig", "gdm", "geiger", 
    "glmgraph", "glmmfields", "growfunctions", "gwsem", "hBayesDM", "hsphase", "hsstan", "iBST", "ibm", "imbibe", "inca", "kergp", "lpme", "mmand", "mmpca", "motmot", "mumm", "mvnfast", "orQA", "pGPx", "parglm", "pdftools", "pkg.name", "psgp", "revealedPrefs", "rococo", "sequences", "spp", "survHE", "synlik", "tmg", "unmarked", "vennLasso", "visit", "wingui", "wsrf", "zic")
Count No Rcpp exports file - 
212
> 


