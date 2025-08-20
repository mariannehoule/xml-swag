```@meta
EditURL = "../data-all-mammals.jl"
```

````@example data-all-mammals
########################################
````

          ALL MAMMALS               #

````@example data-all-mammals
########################################
````

August 2025
At first, I was using HanBat-traits which was a previously cleaned dataset from virionette github repository containing pantheria + elton traits
But then I started working on all mammals, so I needed the complete versions of pantheria and elton traits + complete and updated interaction data from Virion
So this script was born

The older versions of scripts focusing only on bats now needed too much modifications (updating hosts, adding new variables, matching names between datasets, etc.)
So now we're doing data cleaning on all mammals dataset, and then we can filter it to get bats only then do data imputation and the rest

Packages

````@example data-all-mammals
    using Pkg, CSV, DataFrames

#############################
````

MERGE DATASETS

````@example data-all-mammals
#############################
````

Read files

````@example data-all-mammals
    data_path = joinpath(pwd(), "code", "data")
````

pantheria has most of the traits

````@example data-all-mammals
    pantheria = CSV.read(joinpath(data_path, "pantheria-mammals.csv"), missingstring="NA", DataFrame)
````

elton has diet + foraging stratum

````@example data-all-mammals
    elton = CSV.read(joinpath(data_path, "eltontraits.csv"), missingstring="NA", DataFrame)
````

decided to use Virion instead of clover/virionette

````@example data-all-mammals
    virion = CSV.read(joinpath(data_path, "Virion.csv.gz"), missingstring="", DataFrame)
````

Traits data

````@example data-all-mammals
    dropmissing!(pantheria, :MSW05_Genus)

    pantheria.:MSW05_Binomial = uppercasefirst.(pantheria.:MSW05_Binomial)
````

Merge pantheria and elton traits

````@example data-all-mammals
    traits = outerjoin(pantheria, elton, on = :MSW05_Binomial => :Scientific)

    rename!(traits, :MSW05_Binomial => :Host, :MSW05_Order => :Order, :MSW05_Family => :Family, :MSW05_Genus => :Genus, :MSW05_Species => :Species)
````

Update species names by hand; old species names in traits data but new names in virion
Alexandromys oeconomus : formely Microtus oeconomus, which we do have data

````@example data-all-mammals
    replace!(traits.Host,"Microtus oeconomus" => "Alexandromys oeconomus")
````

Clethrionomys rutilus

````@example data-all-mammals
    replace!(traits.Host,"Myodes rutilus" => "Clethrionomys rutilus")
````

Genus Cnephaeus

````@example data-all-mammals
    replace!(traits.Host,"Eptesicus japonensis" => "Cnephaeus japonensis")

    replace!(traits.Host,"Eptesicus serotinus" => "Cnephaeus serotinus")

    replace!(traits.Host,"Eptesicus nilssonii" => "Cnephaeus nilssonii")
````

Dicotyles tajacu

````@example data-all-mammals
    replace!(traits.Host,"Pecari tajacu" => "Dicotyles tajacu")
````

Epomophorus pusillus

````@example data-all-mammals
    replace!(traits.Host,"Micropteropus pusillus" => "Epomophorus pusillus")
````

Laephotis capensis

````@example data-all-mammals
    replace!(traits.Host,"Neoromicia capensis" => "Laephotis capensis")
````

Macronycteris commersonii

````@example data-all-mammals
    replace!(traits.Host,"Hipposideros commersoni" => "Macronycteris commersonii")
````

Macronycteris gigas

````@example data-all-mammals
    replace!(traits.Host,"Hipposideros gigas" => "Macronycteris gigas")
````

Mops plicatus

````@example data-all-mammals
    replace!(traits.Host,"Chaerephon plicatus" => "Mops plicatus")
````

Neogale vison

````@example data-all-mammals
    replace!(traits.Host,"Neovison vison" => "Neogale vison")
````

Xantho

````@example data-all-mammals
    replace!(traits.Host,"Nycticebus pygmaeus" => "Xanthonycticebus pygmaeus")
````

Taeromys dominator

````@example data-all-mammals
    replace!(traits.Host,"Paruromys dominator" => "Taeromys dominator")

### Few species (old names) to figure out
````

Apodemus ilex (rodentia, mulot)

````@example data-all-mammals
    #?
````

Artibeus planirostris
considered under jamaicensis ?

Kerivoula furva
Specimens previously referred to Kerivoula hardwickii?

Submyotodon latirostris
?

Vicugna pacos
Alpaca, considered under Vicugna vicugna?

Interaction data
Delete rows with missing Host

````@example data-all-mammals
    dropmissing!(virion, :Host)

    dropmissing!(virion, :VirusGenus)
````

Only have betacoronaviruses and mammals

````@example data-all-mammals
    interaction = virion[virion.VirusGenus .== "betacoronavirus", :]

    interaction = interaction[interaction.HostClass .== "mammalia", :]
````

Add presence/absence

````@example data-all-mammals
    interaction[!, :betacov] = ifelse.(interaction.VirusGenus .== "betacoronavirus", 1, 0)
````

Unique hosts

````@example data-all-mammals
    interaction = unique(interaction[:, [:Host, :betacov]])

    interaction.Host = uppercasefirst.(interaction.Host)
````

Merge traits and interaction data

````@example data-all-mammals
    allmamms = outerjoin(interaction, traits, on = :Host)

    allmamms[!, :betacov] .= coalesce.(allmamms[!, :betacov], 0)

    #Delete homo sapiens
    allmamms = allmamms[allmamms.Host.!="Homo sapiens",:]
````

Now we have a dataset with all known mammal hosts of betacov and their traits (well almost)

````@example data-all-mammals
##############################################
````

Handle hosts with only missing values

````@example data-all-mammals
##############################################
````

Remove sp.

````@example data-all-mammals
    allmamms = allmamms[allmamms.Host.!="Bandicota sp.",:]
    allmamms = allmamms[allmamms.Host.!="Cynopterus sp.",:]
    allmamms = allmamms[allmamms.Host.!="Plecotus sp.",:]
    allmamms = allmamms[allmamms.Host.!="Rattus sp.",:]
    allmamms = allmamms[allmamms.Host.!="Rhinolophus sp.",:]
    allmamms = allmamms[allmamms.Host.!="Rousettus sp.",:]
````

Bos indicus (only keep bos taurus)

````@example data-all-mammals
    allmamms = allmamms[allmamms.Host.!="Bos indicus",:]
````

Hipposideros cf ruber

````@example data-all-mammals
    allmamms = allmamms[allmamms.Host.!="Hipposideros cf. ruber",:]

### Few species (old names) to figure out
````

Meanwhile, just dropping them

````@example data-all-mammals
    dropmissing!(allmamms, :Order)
````

Extinct species

we have a couple species with absolutely no traits data, after some quick research, they are all extinct species?
so i guess we can just drop them?

drop rows that only have missing values in traits

````@example data-all-mammals
    dropmissing!(allmamms, "Activity-Diurnal")
````

only used column "activiy diurnal" to check for missing values, since elton traits only had missing values for extinct species
then randomly chose this particular column from elton to drop them

````@example data-all-mammals
#############################################
````

     CONVERTING DATA TYPE                 #

````@example data-all-mammals
#############################################
#Before we move on to exclusion criteria (and separate in distinct datasets), let's convert our categorical variables
````

So I don't have to do it later in 3 different sets

````@example data-all-mammals
 using CategoricalArrays
````

Variables catégoriques
Convert target to categorical

````@example data-all-mammals
    allmamms[!, :betacov] = categorical(allmamms[!, :betacov], levels=[0, 1], ordered=true)
````

Convert Terrestriality to categorical

````@example data-all-mammals
    allmamms[!, "12-2_Terrestriality"] = categorical(allmamms[!, "12-2_Terrestriality"])
````

Convert foraging stratum to categorical

````@example data-all-mammals
    allmamms[!, "ForStrat-Value"] = categorical(allmamms[!, "ForStrat-Value"])
````

Convert trophic level to categorical

````@example data-all-mammals
    allmamms[!, "6-2_TrophicLevel"] = categorical(allmamms[!, "6-2_TrophicLevel"])
````

Convert MSWFamilyLatin (including family as an input feature)

````@example data-all-mammals
    allmamms[!, "MSWFamilyLatin"] = categorical(allmamms[!, "MSWFamilyLatin"])
````

if missing dans :MSWFamily, remplacer par valeur dans Family

````@example data-all-mammals
        allmamms[!, :MSWFamilyLatin] = coalesce.(allmamms[!, :MSWFamilyLatin], allmamms[!, :Family])
````

Convert activity

````@example data-all-mammals
    allmamms[!, "Activity-Diurnal"] = categorical(allmamms[!, "Activity-Diurnal"])
    allmamms[!, "Activity-Nocturnal"] = categorical(allmamms[!, "Activity-Nocturnal"])
    allmamms[!, "Activity-Crepuscular"] = categorical(allmamms[!, "Activity-Crepuscular"])
````

List of categorical features

````@example data-all-mammals
    catfeatures = filter(name -> eltype(allmamms[!, name]) <: Union{Missing, CategoricalValue}, names(allmamms[:, 7:end]))

    #Voilà

##############################################
````

CHIRODENTIA

````@example data-all-mammals
##############################################
````

To focus on bats (and rodents)

````@example data-all-mammals
    chirop = allmamms[allmamms.Order .== "Chiroptera", :]

    rodent = allmamms[allmamms.Order .== "Rodentia", :]

    chirodentia = vcat(chirop, rodent)


##########################
````

EXCLUSION CRITERIA

````@example data-all-mammals
##########################
````

Depending on what we're working with, just change chriop/chirodentia/allmamms

````@example data-all-mammals
#EXCLUSION >60% MISSING VALUES
````

Calculate percentage of missing values for each feature

````@example data-all-mammals
    na_percent = map(col -> sum(ismissing, col) / length(col) * 100, eachcol(allmamms))
````

Identify columns with more than 60% missing values

````@example data-all-mammals
    byebye = names(allmamms)[na_percent .> 60]
````

Commentaires post rencontre comité : passer de 50% à 60% missing values pour inclure + variables
À 60% on inclue notamment la "Litter size" -> Han et al 2016:
"We found that filovirus-positive species disproportionately display this tendency to have more than a single litter (pup) per year"
majority of bats have 1 litter per year with a single pup in each litter, but some populations support a second litter in some years
(notably among the Vespertilionidae, the most speciose Family of insectivorous bats, and the Pteropodidae, the Old World fruit bats

Remove those columns

````@example data-all-mammals
    select!(allmamms, Not(byebye))
````

CHIROP : à 60% de missing values, on garde littersize, dietbreadth et trophic level, qui sont autrement retirés à 50%
CHIRODENTIA : 2 variables de plus; habitat breadth et Terrestriality
ALL MAMMS: 4 variables de plus; Littersize, Terrestriality, Diet breadth et Trophic level

Removing other columns we dont want

````@example data-all-mammals
    select!(allmamms, Not([:References, :MSW3_ID]))
````

EXCLUSION plus de 97% heterogene

````@example data-all-mammals
    Pkg.add("StatsBase")
    using StatsBase
````

Fonction calculer la proportion du mode

````@example data-all-mammals
    function mode_prop(x)
        x_no_na = collect(skipmissing(x))
        ux = unique(x_no_na)
        total = countmap(x_no_na)
        max_freq = maximum(values(total))
        proportion = max_freq / length(x_no_na)
        return proportion
    end
````

Specify columns

````@example data-all-mammals
    features = allmamms[:, 7:end]

    #Apply mode_prop to those columns
    homogene = map(col -> mode_prop(allmamms[!, col]), names(features))
````

Find columns where the mode proportion is higher than 97%

````@example data-all-mammals
    exclusion = names(features)[homogene .> 0.90]
````

avec allmamms 97% d'homogénéité ça enlève rien, donc on descend à 90% pour clean up = 6 types de diète sont retirées

Remove those columns from the DataFrame

````@example data-all-mammals
    select!(allmamms, Not(exclusion))
````

CHIROP : 11 features retirées, on termine avec 26 features (critère 97%)
CHIRODENTIA : 4 features retirées, on termine avec 29 features (critère 97%)
ALL MAMMS : 6 features retirées, on termine avec 30 features (critère 90%)

````@example data-all-mammals
### EXPORT DATASETS (juste les features quon utilise)
````

Dataset with all mammals + betacov status + features we're going to use

````@example data-all-mammals
    CSV.write(joinpath(data_path, "allmamms_beta.csv"), allmamms)
    CSV.write(joinpath(data_path, "chirop_beta.csv"), chirop)
    CSV.write(joinpath(data_path, "chirodentia_beta.csv"), chirodentia)


###############################
````

DATA IMPUTATION

````@example data-all-mammals
###############################

using Statistics
using CategoricalArrays

fam_chirop = groupby(chirop, :Family)
fam_chirodentia = groupby(chirodentia, :Family)
fam_allmamms = groupby(allmamms, :Family)
````

Replace missing values in categorical variables with the mode of the family

````@example data-all-mammals
    catfeatures = filter(name -> eltype(allmamms[!, name]) <: Union{Missing, CategoricalValue}, names(allmamms[:, 7:end]))

  for fam in fam_allmamms
       for feature in catfeatures
            colmode = mode(skipmissing(allmamms[!, feature]))
            row_indices = findall(ismissing, allmamms[!, feature])
            for i in row_indices
                allmamms[i, feature] = colmode
            end
        end
    end
````

Replace missing values in numerical variables with the median of the family

````@example data-all-mammals
    numfeatures = filter(name -> eltype(allmamms[!, name]) <: Union{Missing, Number}, names(allmamms[:, 7:end]))
````

This loop kind of works, but we're having problems with families who only have missing values for a feature

````@example data-all-mammals
    for fam in fam_allmamms
        for feature in numfeatures
            colmedian = median(skipmissing(fam[!, feature]))
            coltype = nonmissingtype(eltype(allmamms[!, feature]))
            if coltype <: Int
                colmedian = round(Int, colmedian)
            end
            row_indices = parentindices(fam)[1]
            for i in row_indices
                if ismissing(allmamms[i, feature])
                    allmamms[i, feature] = colmedian
                end
            end
        end
    end
````

Works for some columns, but then i get : ERROR: ArgumentError: median of an empty array is undefined, Float64[]

````@example data-all-mammals
#######################################
````

CHIROP :
Doing it by hand for these families that the loop didnt work for

````@example data-all-mammals
        groupquibug = fam_chirop[("Rhinopomatidae",)]

    for col in numfeatures
        col_median = median(skipmissing(groupquibug[!, col]))
        col_eltype = nonmissingtype(eltype(chirop[!, col]))
        if col_eltype <: Int
            col_median = round(Int, col_median)
        end
        row_indices = parentindices(groupquibug)[1]
        for i in row_indices
            if ismissing(chirop[i, col])
                chirop[i, col] = col_median
            end
        end
    end
````

this works

````@example data-all-mammals
    groupquibug2 = fam_chirop[("Thyropteridae",)]

    for col in numfeatures
        col_median = median(skipmissing(groupquibug2[!, col]))
        col_eltype = nonmissingtype(eltype(chirop[!, col]))
        if col_eltype <: Int
            col_median = round(Int, col_median)
        end
        row_indices = parentindices(groupquibug2)[1]
        for i in row_indices
            if ismissing(chirop[i, col])
                chirop[i, col] = col_median
            end
        end
    end
````

For Noctilionidae, there are only 2 species so median doesnt seem to work?

````@example data-all-mammals
    groupquibug3 = fam_chirop[("Noctilionidae",)]
````

Replace missing values in Nictilionidae with values of row Noctilio albiventris

````@example data-all-mammals
    features = names(chirop[:, 7:end])

    for col in features
        row_indices = parentindices(groupquibug3)[1]
        for i in row_indices
            if ismissing(chirop[i, col])
                chirop[i, col] = chirop[chirop.Host .== "Noctilio albiventris", col][1]
            end
        end
    end
````

this works

Jesus christ there's still one missing value in the dataset

````@example data-all-mammals
    groupquibug4 = fam_chirop[("Myzopodidae",)] #only 1 species in this family
````

replace missing value in litter size with value 1

````@example data-all-mammals
    for i in parentindices(groupquibug4)[1]
        if ismissing(chirop[i, "15-1_LitterSize"])
            chirop[i, "15-1_LitterSize"] = 1
        end
    end

#############################################
````

CHIRODENTIA :

the loop on families is not working for a lot of families and traits :/

````@example data-all-mammals
    order_chirodentia = groupby(chirodentia, :Order)

    for order in order_chirodentia
        for feature in numfeatures
            colmedian = median(skipmissing(order[!, feature]))
            coltype = nonmissingtype(eltype(chirodentia[!, feature]))
            if coltype <: Int
                colmedian = round(Int, colmedian)
            end
            row_indices = parentindices(order)[1]
            for i in row_indices
                if ismissing(chirodentia[i, feature])
                    chirodentia[i, feature] = colmedian
                end
            end
        end
    end
````

Loop on order is working but i think we should find a way to make it work for families

````@example data-all-mammals
############################################
````

ALL MAMMS :

````@example data-all-mammals
    order_allmamms = groupby(allmamms, :Order)
````

Loop for order

````@example data-all-mammals
    for order in order_allmamms
        for feature in numfeatures
            colmedian = median(skipmissing(order[!, feature]))
            coltype = nonmissingtype(eltype(allmamms[!, feature]))
            if coltype <: Int
                colmedian = round(Int, colmedian)
            end
            row_indices = parentindices(order)[1]
            for i in row_indices
                if ismissing(allmamms[i, feature])
                    allmamms[i, feature] = colmedian
                end
            end
        end
    end
````

Same as above, the loop works-ish but then not works for some columns : ERROR: ArgumentError: median of an empty array is undefined, Float64[]

````@example data-all-mammals
#############################################
````

Print missing if there's any missing values in dataset

````@example data-all-mammals
    for col in names(allmamms)
        if any(ismissing, allmamms[!, col])
            println("Missing values found in column: $col")
        end
    end

    ##### Will need a different method for chirodentia and allmamms => median of order for remaining missing values?


######################################
````

EXPORT OUR CLEANED FINAL DATASETS

````@example data-all-mammals
######################################
````

Export cleaned datasets

````@example data-all-mammals
    CSV.write(joinpath(data_path, "chirop_ready.csv"), chirop)
    CSV.write(joinpath(data_path, "chirodentia_ready.csv"), chirodentia)
    CSV.write(joinpath(data_path, "allmamms_ready.csv"), allmamms)
````

---

*This page was generated using [Literate.jl](https://github.com/fredrikekre/Literate.jl).*

