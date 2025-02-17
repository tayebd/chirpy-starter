---
math: true
mathjax: true
layout: post
title:  "Solar PV Design"
array_power: 3045
array_power_kw: '3,045'
nbr_panels: 7
inverter_brand: Growatt
inverter: MIN 6000TL-X
inverter_power: 5000
inverter_min_in_voltage: 80
inverter_MPPT_nbr: 2
inverter_max_in_voltage: 500
inverter_Udcmac: 500
inverter_Umpptmin: 80
inverter_Umpptmax: 500
inverter_start_voltage: 100
inverter_MPPT_voltage_range: 80-500
inverter_apparent_power: 5000
inverter_max_out_current: 22.7
inverter_max_in_current: 13.5
inverter_max_Isc: 16.9
panel_brand: LONGI SOLAR
panel: LR4-72HBD-435M
panel_Pmax: 670 {% Rated Maximum Power %}
panel_Vmp: 38.5
panel_Imp: 17.43
panel_Voc: 46.30
panel_Isc: 18.55 
panel_a_Isc: 0.048 
panel_B_Voc: -0.27   
panel_γ_Pmp: -0.35 
DC_lightning_brand: KONTRON
DC_lightning: PDC2-40 DC 600V
AC_lightning_brand: KONTRON
AC_lightning: PAC2-40 AC 275V
circuit_breaker_brand: SIAME
circuit_breaker: SIAME 2x16 30mA
T_min: -10
customer: Leonardo Borji
---

{%- comment -%} Temperature differences constants {%- endcomment -%}
{% assign temp_diff_cold = -35 %}
{% assign temp_diff_hot = 60 %}

{%- comment -%} Vmp at -10°C calculation {%- endcomment -%}
{% assign beta_factor = page.panel_B_Voc | divided_by: 100.0 %}
{% assign temp_effect_cold = temp_diff_cold | times: beta_factor %}
{% assign temp_multiplier_cold = 1 | plus: temp_effect_cold %}
{% assign Vmp_10 = page.panel_Vmp | times: temp_multiplier_cold %}

{%- comment -%} Voc at -10°C calculation {%- endcomment -%}
{% assign temp_effect_voc_cold = temp_diff_cold | times: beta_factor %}
{% assign temp_multiplier_voc_cold = 1 | plus: temp_effect_voc_cold %}
{% assign Voc_10 = page.panel_Voc | times: temp_multiplier_voc_cold %}

{%- comment -%} Nsoptimal calculation {%- endcomment -%}
{% assign Nsoptimal_div = page.inverter_Umpptmax | divided_by: Vmp_10 | times: 1.0 %}
{% assign Nsoptimal = Nsoptimal_div | floor %}

{%- comment -%} Nsmax calculation {%- endcomment -%}
{% assign Nsmax_div = page.inverter_Udcmac | divided_by: Voc_10 | times: 1.0 %}
{% assign Nsmax = Nsmax_div | floor %}

{%- comment -%} Vmp at 85°C calculation {%- endcomment -%}
{% assign temp_effect_hot = temp_diff_hot | times: beta_factor %}
{% assign temp_multiplier_hot = 1 | plus: temp_effect_hot %}
{% assign Vmp_85 = page.panel_Vmp | times: temp_multiplier_hot %}

{%- comment -%} Nsmin calculation {%- endcomment -%}
{% assign Nsmin_div = page.inverter_Umpptmin | divided_by: Vmp_85 | times: 1.0 %}
{% assign Nsmin = Nsmin_div | ceil %}

{%- comment -%} Isc at 85°C calculation {%- endcomment -%}
{% assign alpha_factor = page.panel_a_Isc | divided_by: 100.0 %}
{% assign temp_effect_isc = temp_diff_hot | times: alpha_factor %}
{% assign temp_multiplier_isc = 1 | plus: temp_effect_isc %}
{% assign Isc_85 = page.panel_Isc | times: temp_multiplier_isc %}

{%- comment -%} Npmax calculation {%- endcomment -%}
{% assign Npmax_div = page.inverter_max_Isc | divided_by: Isc_85 | times: 1.0 %}
{% assign Npmax = Npmax_div | floor %}

{%- comment -%} Imp at 85°C calculation {%- endcomment -%}
{% assign temp_effect_imp = temp_diff_hot | times: alpha_factor %}
{% assign temp_multiplier_imp = 1 | plus: temp_effect_imp %}
{% assign Imp_85 = page.panel_Imp | times: temp_multiplier_imp %}

{%- comment -%} Npoptimal calculation {%- endcomment -%}
{% assign Npoptimal_div = page.inverter_max_in_current | divided_by: Imp_85 | times: 1.0 %}
{% assign Npoptimal = Npoptimal_div | floor %}

{%- comment -%} Power compatibility check {%- endcomment -%}
{% assign total_panels = Nsoptimal | times: Npoptimal %}
{% assign total_power = total_panels | times: page.panel_power %}
{% assign power_ratio = total_power | divided_by: page.inverter_power | times: 1.0 %}
{% assign power_check_min = power_ratio | at_least: 0.9 %}
{% assign power_check_max = power_ratio | at_most: 1.3 %}
{% assign power_compatible = power_check_min == power_ratio and power_check_max == power_ratio %}

### Temperature Calculations
Vmp (à -10°C) = {{ Vmp_10 | round: 2 }} V
Voc (à -10°C) = {{ Voc_10 | round: 2 }} V
Vmp (à 85°C) = {{ Vmp_85 | round: 2 }} V
Isc (à 85°C) = {{ Isc_85 | round: 2 }} A
Imp (à 85°C) = {{ Imp_85 | round: 2 }} A

### Panel Configuration
Nombre optimal de panneaux en série (Nsoptimal) = {{ Nsoptimal }}
Nombre maximal de panneaux en série (Nsmax) = {{ Nsmax }}
Nombre minimal de panneaux en série (Nsmin) = {{ Nsmin }}
Nombre maximal de chaînes en parallèle (Npmax) = {{ Npmax }}
Nombre optimal de chaînes en parallèle (Npoptimal) = {{ Npoptimal }}

### Power Compatibility
Rapport de puissance = {{ power_ratio | round: 2 }}
Compatibilité: {% if power_compatible %}Compatible{% else %}Non Compatible{% endif %}


**Application numérique**
Vmp (à − 10°C) = {{ page.panel_Vmp }} × (1 + {{ page.panel_B_Voc }} / 100 × (-35)) = {{ Vmp_10 }} V

Nsoptimal = E⁻({{ page.inverter_Umpptmax }} / {{ Vmp_10 }}) = {{ Nsoptimal_calc }}

Voc (à − 10°C) = {{ page.panel_Voc }} × (1 + {{ page.panel_B_Voc }} / 100 × (-35)) = {{ Voc_10 }} V

Nsmax = E⁻({{ page.inverter_Udcmac }} / {{ Voc_10 }}) = {{ Nsmax_calc }}

Vmp (à 85°C) = {{ page.panel_Vmp }} × (1 + {{ page.panel_B_Voc }}/100 × 60) = {{ Vmp_85 }} V

Nsmin = E⁺({{ page.inverter_Umpptmin }} / {{ Vmp_85 }}) = {{ Nsmin_calc }}

Isc (à 85°C) = {{ page.panel_Isc }} × (1 + {{ page.panel_a_Isc }}/100 × 60) = {{ Isc_85 }} A

Npmax = E⁻({{ page.inverter_max_Isc }} / {{ Isc_85 }}) = {{ Npmax_calc }}

Imp (à 85°C) = {{ page.panel_Imp }} × (1 + {{ page.panel_a_Isc }}/100 × 60) = {{ Imp_85 }} A

Npoptimal = E⁻({{ page.inverter_max_in_current }} / {{ Imp_85 }}) = {{ Npoptimal_calc }}




Mémoire descriptif: {{ Vmp_10}}
Etude de dimensionnement du système photovoltaïque
Plan d'implantation de l'installation photovoltaïque
Câblage des panneaux Photovoltaïques
Schéma unifilaire des différents composants et modules photovoltaïques
Mise à la terre des panneaux photovoltaïques
Mise en place des outils de sécurité électrique pour l'installation
Calcul de chute de tension
Simulation par logiciel de gisement solaire PVGIS
Plan de signalisation
Maintenance du système photovoltaïque
Certificat d'homologation des modules photovoltaïques
Notices techniques de tous les équipements et accessoires qui seront installés


# Introduction
Le projet consiste à la mise en place d'une installation photovoltaïque de puissance {{ page.array_power_kw }} KWc dans les conditions optimales. 
Dans ce cadre nous allons élaborer les thèmes suivants:
- Dimensionnement d'une installation photovoltaïque.
- Conception du champ photovoltaïque.
- Mise en place des outils de protection électrique pour les déférentes parties de l'installation.
(Descriptif du projet, date prévisionnelle de mise en service)


# Documentation de la Solution proposée


|Désignation | Annexe |
|--------------------------------------- | ------ |
| Dossier administratif<br>(CIN, Registre de commerce, Contrat BT, derniére facture payée)| N°1    |
| Simulation PVsyst ou équivalent        | N°2    |
| Attestation de conformité d’un prototype de la structure delivrée<br>par un bureau de controle| N°3    |
| Schéma unifilaire détaillé en format A3 <br>(à mentionner longueurs, sections cables, caractéristiques techniques équipement et mise à la terre) | N°4    |
| Plan d’implantation | N°5    |
| Plan de situation   | N°6    |
| schéma de câblage des panneaux et disposition des chaines| N°7    |
| schéma de câblage des coffrets DC| N°8    |
| schéma de câblage des coffrets AC| N°9    |
| Homologation de l’onduleur| N°10   |



# Equipements de la Solution proposée

Dans le but d'assurer la production d'énergie électrique a partir des énergies renouvelables pour l'autoconsommation et vente de l'excédent, 
{{ page.customer }} va effectuer une installation photovoltaïque d'une puissance égale à {{ page.array_power_kw }} KWc. 
Cette puissance est assurée à l'aide de {{ page.nbr_panels }} modules de type {{ page.panel_brand }}, 
dont la puissance d'un seul module est égale à {{ page.panel_power }} Wc. L'ensemble des modules sera associé à un onduleur, 
{{ page.inverter }}, afin de fournir une tension alternative. Le système photovoltaïque comporte aussi 
deux coffrets de protection (un coffret de protection DC et un coffret de protection AC), afin de garantir 
la protection électrique du coté générateur photovoltaïque d'une part et du coté AC de l'autre part selon le guide:


UTE 15-712-1 et le cahier de charge STEG pour les connections réseau. 
L'installation est composée des éléments cités dans le Tableau 1. 
Les fiches techniques de chaque élément figurent dans l'annexe à la fin de ce rapport.


| Désignation   | Nombre | Marque        | Référence        | Annexe |
| --------------| ------ | ------------- | ---------------- | ------ |
| Modules       | 7| {{ page.panel_brand }}   | {{ page.panel }}  | N°11    |
| Onduleur      | 1 | {{ page.inverter_brand }}  | {{ page.inverter }} | N°12    |
| Fusibles      | 1| {{ page.DC_lightning_brand }}       | {{ page.DC_lightning_brand }}  | N°13    |
| Parafoudre DC | 1| {{ page.DC_lightning_brand }} | {{ page.DC_lightning_brand }}  | N°14    |
| Interrupteurs Sectionneur DC| 1| MERZ| MDC10-040-1000-2 | N°15    |
| Parafoudre AC | 1| {{ page.AC_lightning_brand }}| {{ page.AC_lightning_brand }}| N°16|
| Interrupteur sectionneur général | 1|{{ page.circuit_breaker_brand }}| {{ page.circuit_breaker }}  | N°17 |
| Disjoncteur différentiel AC 30mA| 1  | {{ page.circuit_breaker_brand_AC_30 }}| {{ page.circuit_breaker_AC_30 }}  | N°18 |
| Disjoncteur général AC | 1 | {{ page.circuit_breaker_brand_AC }} | {{ page.circuit_breaker_AC }} | N°19    |
| Câble DC| 1 | EXTRA CABLE   | Section 4 mm2    | N°20    |
| Câble AC| 1 | Tunisie câble | 2*4 mm2          | N°22    |
| Cable de mise à la terre| 1 | Tunisie câble | 2*4 mm2| N°23|
| Connecteur MC4| 1 | Multi contact | MC4 | N°24    |
| Répartiteurs| 1 | Multi contact | MC4 | N°25    |
| Chemin de câble|1| Multi contact | MC4              | N°26    |

**Tableau 1: Principaux Composants de l'Installation**


# Equipements de la Solution Proposée 


## Caractéristiques Techniques des équipements choisis

### Panneau 

| Marque | {{ page.panel_brand }}  |       
| -- | -- |
| Référence                         | {{ page.panel }}   |
| Puissance unitaire (W) (STC)                         | {{ page.panel_power }}   |
| Vmpp (STC)              | {{ page.panel_Vmp }} |
| Impp (STC)              | {{ page.panel_Imp }} |
| Voc (STC)                 | {{ page.panel_Voc }}   |
| Isc(STC)                 | {{ page.panel_Isc }}  |
| Coefficient de température Voc (STC)  | {{ page.panel_B_Voc}}  |
| Coefficient de température Isc (STC)  | {{ page.panel_a_Isc}}  |
| I~RM~  |   |



### Certifications et garanties
Les modules {{ page.panel_brand }} ont plusieurs certifications: IEC61215, IEC61730, UL 61730, ISO9001:2015, ISO14001:2015, ISO45001:2018, TS62941.

Garantie de rendement: 12 ans pour le produit et 30 ans de garantie linéaire.


## Onduleur

#### Onduleur N°1

| Numéro onduleur|Nombre panneaux|Puissance crête DC|Rapport de puissance|
| -- | -- | -- | -- |
| 1  | A fractionner la cellule selon nombre MPPT et chaines/MPPT  |   |   |



| Onduleur 1| Onduleur 1  |
| -- | -- |
| Marque  | {{ page.inverter_brand }}  |
| Référence  | {{ page.inverter }}  |
| Puissance AC(W)  | {{ page.inverter_power}}  |
| Vdcmax (V)  |{{ page.inverter_max_in_voltage }}   |
| Idcmax/MPPT (A)  | {{page.inverter_max_in_current}}   |
| Iscmax /MPPT (A)  | {{page.inverter_max_Isc}}   |
| Nb MPPT  | {{ page.inverter_mppts }}  |
| Nb entrées/MPPT  | 1   |
| Plage MPPT (V)  | {{page.inverter_MPPT_range }}  |
| Plage de tension d’entrée (V)  | {{page.inverter_MPPT_voltage_range}}  |
| Puissance AC (kVA)  | {{page.inverter_apparent_power}}  |
| Tension de sortie (V)  | {{page.inverter_voltage}}   |
| Iac max(A)  |{{ page.inverter_max_out_current }}    |


### Caractéristiques équipements DC et AC 

| Equipement  | Caractéristiques techniques  |
| -- | -- |
|  Fusibles  | U~fusible~ =<br>I~fusible~ = |
|  Parafoudre DC  | Type 1 ou 2<br>Ucpv =<br>Up = <br>In = <br>Iscpv =    |
| Interrupteur sectionneur DC  | Usec =<br>Isec =  |
|  Disjoncteur (différentiel) AC  |  Udis =<br>In =<br>Pouvoir de coupure =<br>Sensibilité =  |
|  Parafoudre AC  | Type 1 ou 2<br>Ucpv =<br>Up = <br>In = <br>Iscpv =   |
|  Section câbles DC  | Section<br>Courant admissible Iz=  |
|  Section câbles AC  | Section<br>Courant admissible Iz=  |


### Compatibilité de l’onduleur

| Valeur  | Description  |Valuer|
| -- | -- |---|
|T~min~| Température minimale |-10°C
|T~max~|Température maximale |85°C
|𝜷|coefficient tension/température du module photovoltaïque (%/°C)|{{ page.panel_B_Voc }}|
|α|coefficient courant/température du module photovoltaïque (%/°C)|{{ page.panel_a_Isc }}|
|U~mpptmax~|Tension maximale de la plage MPPT de l’onduleur|{{page.inverter_max_in_voltage}}|
|U~mpptmin~|Tension minimale de la plage MPPT de l’onduleur|{{page.inverter_min_in_voltage}}|
|I~max~|Courant d’entrée maximal par MPPT|{{page.inverter_max_in_current}} |
|I~cc~|Courant de court circuit de l’onduleur par MPPT|{{page.inverter_max_Isc}} |
|I~sc~|Courant de court circuit du panneau|{{page.panel_Isc}}|


#### Nombre maximal de panneau en série (protection onduleur – champ PV)

Nsmax = $E ^ { - } ( \frac { Udcmax } { Voc ( à  -10 ^ { \circ } C ) } )$

Voc (à − 10°C) = Voc * ( 1 + 𝛽/100 * (-10 - 25°C)) ; 𝛽 en %/°C et Tmin =-10°C

Udcmax : tension d’entrée maximale de l’onduleur

**Application numérique**

Npv = Pc/Ppv = {{ page.array_power }} / {{ page.panel_power }}  ~  {{ page.nbr_panels }} panneaux.

#### Nombre optimal de panneau en série

Nsoptimal = $E ^ { - } ( \frac { Umpptmax } { Vmp ( à \space -10 ^ { \circ } C ) } )$

Vmp (à − 10°C) = Vmp * (1+ 𝛽/100 * (-10 - 25°C)) ; 𝛽 en %/°C et Tmin =-10°C

**Application numérique**
Vmp (à − 10°C) = Vmp * (1+ 𝛽/100 * (-35)) = 
{% assign Vmp_10 = '377,86' %}


#### Nombre minimal de panneau en série

Nsmin = $E ^ { + } ( \frac { Umpptmin } { Vmp ( à  \space 85 ^ { \circ } C ) } )$

Vmp (à 85°C) = $Vmp*(1+\frac{\beta}{100}*(85-25^{\circ}C))$; $\beta$ en %/°C et Tmax = 85°C

**Application numérique**

#### Nombre maximal de chaines en parallèle (protection - cas cc)
Npmax = $E ^ { - } ( \frac { Icconduleur } { Isc ( a \space 85 ^ { \circ } C ) } )$

Isc (à 85°C) = $Isc*(1+\frac{\alpha}{100}*(Tmax-25^{\circ}C))$; $\alpha $ en %/°C et Tmax = 85°C

**Application numérique**

#### Nombre optimal de chaines en parallele

Npoptimal = $ E ^ { - } ( \frac { Imax } { Imp ( a \space 85 ^ { \circ } C ) } )$

Imp (à 85°C) = $Imp*(1+\frac{\alpha}{100}*(Tmax-25^{\circ}C));\alpha$ en %/°C et Tmax = 85°C

**Application numérique**

#### Compatibilité en puissance
Les puissances du générateur photovoltaïque et de l’onduleur doivent être mutuellement accordées. Le rapport entre La puissance du générateur photovoltaïque et la puissance nominale AC des onduleurs doit être compris entre 0.9 et 1.3.

$0 . 9 \leq \frac { Pcpv } { Pac \space ond } \leq 1 . 3$

**Application numérique**


### Dimensionnement Dispositifs de protection coté DC:
#### Nombre maximal de chaînes en parallèle sans protection
<!--- <p style="text-align:center;">TNcmax ≤ (1+IRM/Isc-STC)</p> ---->

$$TNcmax ≤ (1+\frac{I_{RM}}{I_{scSTC}})$$

#### Nombre maximal de chaînes en parallèle par dispositif de protection
Npmax ≤ 0.5(1+Irm/Isc max)

**Application numérique**

#### Fusible DC
##### Exigences:
- Tension assignée d ’emploi : Ufusible ≥ Vocmax (-10°C)
- Courant assigné 1.1 Iscmax = 1.1 \* 1.25 IscSTC ≤ Ifusible ≤Irm

**Application numérique**
In fusible utilisé =  (A)
Un fusible utilisée = (V)

Fabricant du fusible choisi: 
Référence et type:

#### Interrupteur sectionneur DC
Exigences:
- Tension de l’interrupteur sectionneur Usec > Voc(-10) champ PV
- Courant de l’interrupteur sectionneur Isec> 1,25 x Isc champ PV
    (cas bifacial Isec > Isc avec gain) champ PV 

**Application numérique**
I~intersect~ utilisé =  (A)
U~intersect~ utilisée =  (V)

Fabricant du l’intersec choisi: 
Référence et type:

#### Parafoudre DC
Exigences :
- Type I ou type II
- Tension maximale de régime permanent Ucpv > Uoc max = 1.2 UocSTC
- Niveau de protection d’un parafoudre Up < 80 % de la Tension de tenue aux chocs des équipements (modules, onduleurs) et (Up < 50% pour les équipements distants de plus de 10 mètres)
- Courant nominal de décharge In > 5 kA
- Courant de tenue en court-circuit Iscpv > Iscmax PV = 1.25 IscSTC

**Application numérique**
Ucpv utilisé =
Up =
In utilisé =
Iscpv utilisé =

Fabricant du parafoudre choisi: 
Référence et type:

#### Dimensionnement Dispositifs de protection coté AC
#### Disjoncteur (différentiel) AC
Exigences :
- Tension assignée d’emploi Ue= 230 V ou 400 V en général
- Imax onduleur ≤ le courant d’emploi (courant assigné) Ie ≤ I~z_cable_AC~
- Sensibilité : 30 mA (applications domestiques)

**Application numérique**
Udis =
In disj =
Sensibilité disj =

Fabricant du disjoncteur AC choisi: 
Référence et type:

#### Parafoudre AC
Exigences :
- Type I ou Type II
- Tension maximale de régime permanent Uc > 1.1*(Ue=230)
- Niveau de protection d’un parafoudre Up < 80 % de la tension de tenue aux chocs des équipements à protéger et (Up < 50% pour les équipements distants de plus de 10mètres)
- Courant nominal de décharge In > 5 Ka

**Application numérique**
Uc utilisé =
In utilisé =

Fabricant du parafoudre AC choisi: 
Référence et type:

#### Interrupteur sectionneur général AC

Exigences :
- Tension de l’interrupteur sectionneur Usec >= Uond
- Courant de l’interrupteur sectionneur Isec > nbre onduleur x Imax

**Application numérique**

Fabricant de l’interrupteur AC choisi: 
Référence et type:


### Dimensionnement Câble DC/AC
Exigences :
- La température ambiante utilisée pour dimensionner les câbles DC/AC :
- Enterré : 25 °C
- Dans un Local technique : 50 °C
- Dans un chemin de câble non exposé au soleil : 50 °C
- Dans un chemin de câble exposé au soleil : 80 °C

#### Câbles DC
Le choix du courant admissible Iz des câbles de chaînes PV doit tenir compte des différents facteurs de correction définis dans l’NF C 15-100.

Courant admissible :
Iz’ = Iz \* (K1 \* K2 \* K3 \* K4) A calculer

Avec :

| Valeur  | Description  |
| -- | -- |
Iz’|Courant maximum admissible du câble choisi en tenant compte des conditions de pose
Iz|Courant admissible du câble choisi
K1|Facteur de correction prenant en compte le mode de pose
K2|Facteur de correction prenant en compte l’influence mutuelle des circuits placés côte à côte (groupement de circuits)
K3|Facteur de correction prenant en compte la température ambiante et la nature de l’isolant
K4|Facteur de correction prenant en compte le nombre de couches de câble dans un chemin de câbles

A spécifier les tronçons considérés pour le calcul.

#### Modes de pose
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Méthodes de référence
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Groupement de circuits
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Facteurs de correction pour pose en plusieurs couches
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Température ambiante
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Ombre de couches
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Résistivité thermique de sol
A décrire et à joindre le tableau de la norme avec indication sur le tableau (En cas de câbles enterrés)

NB : Néanmoins, toute autre disposition mentionnée à la norme NC 15-100 est applicable

Conclusion :

| Courant admissible du câble DC | Section | Courant admissible Iz | Courant corrigé Iz’ |
| -- | -- | -- | -- |
|10| 6mm2| 10  | 10 |


**Calcul de chute de tension DC**

Δu(en %) = 100 *  ∆u/Ump 

$Δu = 100 *  \frac {∆u}{Ump}$ (en %)

La chute de tension totale est limité à 3% .

$\rho$ = 0,02314 Ωmm²/m pour le cuivre et $\rho$ = 0,037 Ωmm²/m pour l'aluminium 


La chute de tension doit se calculer pour tous les tronçons.

Tableau à modifier selon la configuration de l’installation

|     | **ρ (Ωmm²/m)** | **L(m)** | **I(A)** | **Section(mm²)** | **V** | 𝚫𝐮 | 𝚫𝐮 **%** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Ch PV 1_ond1 |     | …   | …   | …   | …   | …   | …   |
| Ch PV 2_ond1 |     |     |     |     |     |     |     |

|     | **ρ** | **L** | **I** | **Section** | **V** | 𝚫𝐮 | 𝚫𝐮 **%** |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Ch PV 6_ond2 |     | …   | …   | …   | …   | …   | …   |
| Ch PV 7_ond2 |     |     |     |     |     |     |     |

Conclusion :


- **Caractéristiques des câbles DC minimales**

Le câble doit avoir les caractéristiques techniques minimales suivantes :

- Type de câble : unipolaire, double isolation, résistant aux ultraviolets ;
    - Section des câbles : normalisée
    - Respect des normes des câbles pour courant continu

Ainsi, il est prévu que :

- La température maximale admissible sur l’âme en régime permanent est de 90°C ou 120°C. (isolation PRC)
    - La température maximale admissible sur l’âme en régime de court-circuit est de 250°C.
    - Tension maximale en courant continu : 1,8 kV.
    - Tension assignée en courant alternatif : U0/U : 0,6/1 (1,2) kV
        - U0 : la valeur efficace entre l'âme d'un conducteur
        - U : la valeur efficace entre les âmes conductrices de deux conducteurs



### 4. Chute de tension AC

Le choix du courant admissible Iz des câbles AC doit tenir compte des différents facteurs de correction définis dans l’NF C 15-100.

Courant admissible :
Iz’ = Iz \* (K1 \* K2 \* K3 \* K4) A calculer

Avec :

| Valeur  | Description  |
| -- | -- |
Iz’|Courant maximum admissible du câble choisi en tenant compte des conditions de pose
Iz|Courant admissible du câble choisi
K1|Facteur de correction prenant en compte le mode de pose
K2|Facteur de correction prenant en compte l’influence mutuelle des circuits placés côte à côte (groupement de circuits)
K3|Facteur de correction prenant en compte la température ambiante et la nature de l’isolant
K4|Facteur de correction prenant en compte le nombre de couches de câble dans un chemin de câbles

A spécifier les tronçons considérés pour le calcul.

#### Modes de pose
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Méthodes de référence
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Groupement de circuits
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Facteurs de correction pour pose en plusieurs couches
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Température ambiante
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Ombre de couches
A décrire et à joindre le tableau de la norme avec indication sur le tableau

### Résistivité thermique du sol
A décrire et à joindre le tableau de la norme avec indication sur le tableau (En cas de câbles enterrés)

NB : Néanmoins, toute autre disposition mentionnée à la norme NC 15-100 est applicable

Conclusion :

| Courant admissible du câble AC | Section | Courant admissible Iz | Courant corrigé Iz’ |
| -- | -- | -- | -- |
|10| 6mm2| 10  | 10 |



#### Chute de tension AC

La chute de tension doit se calculer pour tous les tronçons jusqu’au point d’injection.

∆u = b \* ( 𝛒 \* L/S * cos(φ) + λ * L + sin(φ)  ) \* I~maxond~
∆u ( en % ) = 100 \*  ∆u/V~e~

$$\Delta u = b ( \rho \times \frac {L}{S} \times \cos \varphi + \lambda * L + \sin \varphi ) \cdot I _ {maxond}$$
$\Delta u ( e n \% ) = 1 0 0 \times \frac { \Delta u } { V e }$

La chute de tension totale est limitée à 3%.

b = 1 pour les circuits triphasés et b = 2 pour les circuits monophasés

$\rho$ = 0,02314 Ωmm²/m pour le cuivre et $\rho$ = 0,037 Ωmm²/m pour l'aluminium 


|                         | 𝛒 Ωmm²/m | L(m) |I(A)| Section(mm2)| ∆u  | Δu (en %) |
| ---                     | ---      | ---  | --- | ---       |--     | ---|
| Onduleur --- coffret AC |          |     |     |            |  |    |  |
| Coff AC ---- TGBT       |          |     |     |            |  |    | |


- b = 2
- λ = 0,00008
- cos(φ) = 0,8
- sin(φ) = 0,6
- V = 230

**Conclusion**

∆utot = ∆u1 + ∆u2 avec

- ∆u1 est la chute de tension entre onduleur et coffret Ac et

- ∆u2 est la chute de tension entre coffret AC et TGBT

# Description du câblage des panneaux et de la mise à la terre

**Le câblage des panneaux doit se faire conformément au référentiel STEG afin de minimiser la boucle d’induction**

**A Choisir la section des câbles de mise à la terre conformément au référentiel STEG**

# Description de la mise en œuvre de la structure

**A décrire**

**Prévoir une attestation de conformité pour un prototype fournit par un bureau de contrôle que la structure supporte les charges de l’IPV et un vent de 120km/h (note de calcul et construction)**

# Système de comptage

**ANNEXE IPV**

A suivre l’ordre du (tableau I) pour la mise de la documentation des annexe











# Conception du champ photovoltaïque

Pour déterminer le nombre de panneaux nécessaire de cette installation on calcule le rapport de la puissance crête total sur la puissance maximale du panneau selon l'équation suivante :

Npv: nombre de panneaux

Npv = Pc/Ppv

Ppv: puissance maximale d'un panneau

Pc: puissance totale du générateur photovoltaïque

Npv = Pc/Ppv = {{ page.array_power }} / {{ page.panel_power }}  ~  {{ page.nbr_panels }} panneaux.


### Vérification d'intensité d'entrée maximale DC

Isc = {{ page.panel_Isc }} < Iinput onduleur = 14 A vérifié✔

Vérification de tension d'entrée maximale DC à -10 °C:

#### Calcul de Vocmax

Vocmax est la tension maximale aux bornes du module PV à vide se calcule avec la formule :

Vocmax = Ku max * Vocstc

Le facteur de correction Ku prend en compte l'augmentation de la tension en circuit ouvert des modules, en considèrant la température ambiante minimale Tmin du site d'installation PV et le coefficient de variation de la tension du module en température **α**Voc.

Ku = 1 + (**α** * Voc / 100) * ( Tmin - 25 )

Ku = 1 + (-0,284 / 100) * ( -10 - 25 )

Ku = 1,0994

Vocmax = Ku * Vocstc

Vocmax = 1,0994 * {{ page.panel_Voc }} = 53,98 V
{% assign Vocmax = '53,98' %}

La tension d'entrée maximale DC à -10 °C :

Vmax = {{ page.nbr_panels }} * Vocmax = {{ page.nbr_panels }} * {{ page.PV_Vocmax }} = 377,86 V
{% assign Vmax = '377,86' %}

Vmax = {{ page.Vmax }}  < {{ page.inverter_max_in_voltage }} V (tension d'entrée max de l'onduleur) vérifié✓

Vérification de tension d'entrée minimale DC à 85 °C:

#### Calcul de Vmp minimale

Vm minimale est la tension minimale aux bornes du module PV en charge se calcule avec la formule:

Vmp min = Ku * Vmpstc

Le facteur de correction Ku prend en compte l'augmentation de la tension en circuit fermé des modules, en considèrent la température ambiante maximale Tmax du site d'installation PV et le coefficient de variation de la tension du module en température aVmp.

Ku = 1 + ( aVmp / 100 ) * ( Tmax - 25 )

Ku = 1 + ( -0,284 / 100 )  * ( 85 - 25 )

Ku = 0,8296

Vmp min = Ku * Vmpstc

Vmp min = 0,8296 * {{ page.panel_Vmp}} - 33,84 V
{% assign PV_Vmpmin  = '33,84' %}

La tension d'entrée minimale DC à 85°C:

Vmpmin = {{ page.nbr_panels }} * Vmp min = {{ page.nbr_panels }} * {{ page.PV_Vmpmin }} = 236,88 V
{% assign Vmpmin  = '236,88' %}

Vmp min = {{ page.Vmpmin }} V >  {{ page.inverter_min_in_voltage }} ( tension d'entrée minimale de l'onduleur ) vérifié ✔

La compatibilité champ photovoltaïque et onduleur:

La tension varie avec l'éclairement ( 50 à 1000 W/m2 ) et la temperature des cellules PV ( -10°C à 85°C )

![[Pasted image 20250210133225.png]]

### Courbe charactéristique d'un module PV


A Tmin: VocmaxPV < Vondmax de l'onduleur

A Tmax: Vmpp min PV > Vondmin de l'onduleur

Il faut s'assurer aussi que le courant débité par le champ PV ne dépasse pas la valeur du courant maximal 
admissible Imaxpar l'onduleur ( ou chaine vers MPPT onduleur )

ImppSTCPV <  Imax entrée onduleur


Calcul des valeurs de tension aux différentes températures pour un seul module:

| Grandeur mesurée, condition standard de test STC | Symbole| Valeur STC | Coefficient de température | Valeur à Températur eT = -10 C° | Valeur à Température T = 85 C° |
| --------------------------------------------- | --------| ---------- | -------------------------- | ------------------------------- | ------------------------------ |
| Puissance maximale|Pmpp|{{ page.panel_power }} Wc|-0,35 %/C|488,2 W|343,65 W|
| Tension à la puissance maximale|Vmpp| {{ page.panel_Vmp}} V| -0,284 %/C|44,85 V| {{ page.PV_Vmpmin }} V|
| Courant à la puissance maximale|Impp| {{ page.panel_Imp }} A | 0,050 %/C| 10,47 A| 10,97 A|
| Courant de court-circuit| Isc| {{ page.panel_Isc }} A| 0,050 %/C| 11,16 A|11,70 A|
| Tension circuit ouvert | Voc| {{ page.panel_Voc }} V| {{ page.PV_Vocmax }} V | 40,73 V |-0,284 %/C|



| Grandeur mesurée, condition standard de test STC  |Symbole| Valeur à Température T = -10C° | Valeur à Température T = 85C° |
| ---------------------------------------------------------|----|------------------------------- | ------------------------------ |
|Onduleur| Compatibilité tension | {{ page.PV_Vmax }} V  < {{ page.inverter_max_in_voltage }} V vérifié ✔       | {{ page.Vmpmin }} V > {{ page.inverter_min_in_voltage }} vérifié          |
|| Compatibilité courant | 18 A  >  11,1 A vérifié| 18 A  >  11,70 A vérifié ✓       |


# Plan D'implantation de l'Installation Photovoltaïque

![[Screenshot from 2025-02-09 13-15-51.png]]


## Cablage Des Panneaux Photovoltaïques

![[Screenshot from 2025-02-09 13-16-21.png]]


# Schema Unifialire des Differents Composants et Modules Photovoltaïques

![[Screenshot from 2025-02-09 13-16-49.png]]


# Mise a la terre des panneaux Photovoltaïques
![[Screenshot from 2025-02-09 13-17-19.png]]


# Mise en place des outils de sécurité électrique pour l'installation

En ce qui concerne la protection des personnes contre les risques électriques, le guide UTEC 15-712 prescrit l'usage de la classe II pour toute la partie DC. Pour une seule chaine et même jusqu'à trois chaines connectées en parallèle à l'entrée d'un onduleur photovoltaïque, la norme ne préconise pas d'autre protection. Cependant, pour des raisons évidentes de maintenance et d'interchangeabilité de l'onduleur, il est obligatoire d'avoir un sectionneur DC en amont de l'onduleur.

Et pour la partie AC de l'installation ce sont les règles de la NF C 15-100 qui doivent être appliqués. 

## Protection coté DC

![[Pasted image 20250210135834.png]]
Figure 1:câblage du coffret DC

Le coffret de protection utilisées est constitué de :

- Un Coffret de 8 modules IP65
- Un parafoudre DC 600V
- Un interrupteur sectionneur DC


Afin de protéger les équipements (modules photovoltaïques et onduleur) contre les coups de foudre

indirects, des parafoudres doivent être installés de part et d'autre de différentes liaisons.

Nombre de pôles du parafoudre : 2 Poles

• Le niveau de protection Vp < 3 KV

• La tension de service Vc: 600 > Vmax * 1.2 = 453,432 V

• Courant: In = 20 kA > 5 Ka

Référence: {{ page.DC_lightning_brand }} PDC2-40 DC 600 V

- Choix de l'interrupteur-sectionneur:

Les caractéristiques de cet interrupteur sectionneur DC sont les suivant :

- Tension assignée d'emploi: 1000 V >  1,2 * Vmax = 453,432 V
- Catégorie d'emploi DC-20A > 1,25 * 1,1 * Isc = 1,25 * 1,1 * {{ page.panel_Isc }} = 15,62 A

Référence: MERZ MDC-040-1000-2

## Protection coté AC

Pour la partie AC de l'installation le coffret de protection comporte un disjoncteur différentiel et un parafoudre AC:

![[Pasted image 20250210140524.png]]


## Choix du parafoudre AC

Le choix du parafoudre AC répond aux conditions suivantes:

- Parafoudre type 2
- Vc: 275 V selon réseau monophasé
- Vp parafoudre < Vw de l'onduleur

Référence: PARAFOUDRE AC {{ page.DC_lightning_brand }} PAC2-40 AC 275V

## Choix du disjoncteur différentiel

- Ve: Tension assignée d'emploi : tension maximale de fonctionnement
- In: Courant assigné ("calibre"): valeur maximum du courant permanent
- Icu: Pouvoir de coupure ultime ou pouvoir de coupure nominal Icn

Référence: SIAME 2*16 30 mA

# Calcul de chute de tension

## Chute de tension coté DC

Ɛ est la chute de tension admissible tolérée

Par définition: **ε** = ( 2 * L * Imax * ρ ) / ( S * Vmax )

Avec **ρ** Résistivité du matériau conducteur (cuivre ou aluminium) en service normal.

Conformément au guide UTE C15-712-1, ρ = 0,02314 est la résistivité du cuivre on exprimera la résistivité en Ω mm2/m.


S: Section du câble, S=4mm2

Imax : courant maximale circulant dans le câble DC, Imax = {{ page.panel_Isc }} A

Vmax : la tension DC maximale. Vmax = {{ page.PV_Vmax }} V

AN:

**ε** = ( 2 * 10 * {{ page.panel_Isc }} * 0,02314)/(4 * {{ page.PV_Vmax }}) = 0,0034 = 0,34% < 3% ce qui est conforme au guide UTE C15-712-1.

## Chute de tension coté AC

Par définition la chute de tension coté AC:

AV  = b * ( ρ1 * L/s ) * Ib

Avec:

b: coefficient qui vaut 1 en triphasé et 2 en monophasé, b = 2

ρl = 1,25 * ρ = 0,0225 Ω mm2/m : résistivité du matériau conducteur (cuivre).

L: longueur du câble AC. L = 10m

S: section du câble AC. s = 4mm2

Ib: courant maximale d'emploi, Ib = 14,3 A

AN:

ΔV = 2 * ( 0,0225 * 10 / 4 )  * 14,3 = 1,6 V

**ε** %= ΔV/V  * 100 = ( 0,8 / 230 ) * 100 = 0,69% < 3% ce qui est conforme au guide UTE C15-712-1

# Simulation par logiciel photovoltaïque PVGIS
![[Screenshot from 2025-02-09 13-20-20.png]]


# Plan de signalisation

Pour des raisons de sécurité on a ajouté des étiquettes de signalisation pour protéger nos clients contre les dangers électrique.

➤ Etiquetage sur la partie AC :

![[Pasted image 20250210142216.png]]

Une étiquette de signalisation située sur le coffret AC assurant la limite de concession.

➤ Etiquetage sur la partie DC:

Toutes les boites de jonction (générateur PV et groupes PV) et canalisation DC devront porter un marquage visible et inaltérable indiquant que des parties actives internes à ces boites peuvent rester sous tension même après sectionnement de l'onduleur coté continu.

![[Pasted image 20250210142416.png]]


➤ Etiquette portant la mention "Attention, câbles courant continu sous tension":

- Sur la face avant des boites de jonction
- Sur la face avant des coffrets DC

![[Pasted image 20250210142640.png]]

➤ Etiquetage sur l'onduleur :

Tous les onduleurs doivent porter un marquage indiquant qu'avant toute intervention, il y a

lieu d'isoler les sources de tension.

![[Pasted image 20250210142709.png]]



# Maintenance du système photovoltaïque

Si la technologie photovoltaïque est réputée fiable et sans entretien lourd, des opérations de

maintenance légères sont tout de même à conduire pour prévenir d'éventuelles anomalies et

s'assurer que les organes de sécurité sont en état de fonctionnement.

➤ L'inspection et le nettoyage des modules:

➤ L'état de fixation des modules par rapport à la structure


➤ Vérification et dépoussiérage des onduleurs (annuel)

Vérification et dépoussiérage des onduleurs tous les ans, c'est-à-dire:

- Vérifier le fonctionnement des onduleurs (Led témoins, affichage sur les appareils...).
    
- Nettoyer si besoin les entées d'air des onduleurs (ventilateurs) et/ou les dissipateurs de ces derniers pour faciliter leur refroidissement.
    
- S'assurer que les dispositifs de ventilation du local recevant les onduleurs sont propres.
    

➤ Inspection des boitiers DC (annuel):

- Vérifier le bon état des isolants et l'absence de dégâts causés par les animaux (rongeurs).
    
- Vérifier le serrage des connexions.
    
- Contrôler l'état des parafoudres (fenêtres d'état sur la protection).  
    
- Contrôler l'état des fusibles
    

➤ Testes électriques (annuel):

- Manœuvrer les protections AC et contrôler le découplage de l'onduleur.
    
- Manœuvrer les protections DC.
    
- Vérifier la continuité des liaisons équipotentielles.  
    
- Mesurer les tensions de branche DC.
    
- Tester les dispositifs d'arrêt d'urgence.
    

# Certificat d'homologation des modules photovoltaïques

# Notices Techniques de Tous les Equipements et Accessoires qui seront Installes

## Annexe N°1. Fiche Technique Modules Photovoltaïques


## Annexe N°2 Fiche Technique Onduleur


## Annexe N°3. Fiche Technique Parafoudre DC
![[Pasted image 20250210143953.png]]


## Annexe N°4. Fiche Technique Sectionneur
![[Pasted image 20250210143756.png]]


## Annexe N°5. Fiche Technique Parafoudre AC
![[Pasted image 20250210143857.png]]

## Annexe N°6. Fiche Technique Disjoncteur Differentiel
![[Pasted image 20250210144236.png]]


## Annexe N°7. Fiche Technique Cable DC

![[Pasted image 20250210144407.png]]

## Annexe N°8. Fiche Technique Cable AC

![[Pasted image 20250210144513.png]]

## Annexe N°9. Fiche Technique Connecteur

![[Pasted image 20250210144607.png]]



