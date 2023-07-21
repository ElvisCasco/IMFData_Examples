# IMFData_Examples

El archivo "IMFData.qmd" contiene algunos ejemplos para sacar el máximo provecho a la librería [IMFData](https://github.com/stephenbnicar/IMFData.jl). Al ejecutar el proceso desde la carpeta descargada, se actualizan los datos de los archivos `.csv`  adjuntos.

## Varios países, un indicador

```
#import Pkg; Pkg.add("IMFData")
using CSV, DataFrames, DataFramesMeta, IMFData

wd = @__DIR__
# cambiar por el directorio donde se quiere guardar los archivos, por ejemplo:
wd = "C:/Directorio_Trabajo/Julia/IMFData"

function get_df(x)
    try
        df = countries_indicators[x].series
        df.country .= countries_indicators[x].area
        df.indicator .= countries_indicators[x].indicator
        df.frequency .= countries_indicators[x].frequency
        return df
    catch
        return DataFrames.DataFrame()
    end
end

indicators = "NGDP_SA_XDC"
countries  = ["US","CA","MX"]
countries_indicators = get_ifs_data(countries, indicators, "Q", 1900, 2100)
size(countries_indicators)[1]
get_df(3)
ndf = size(countries_indicators)[1]
df = map(_ -> DataFrames.DataFrame(), 1:ndf)
df = vcat([df[i] = get_df(i) for i in 1:ndf]...)
CSV.write(
    wd * "/IMFData.csv",
    delim = ";",
    df)
```

## Varios países, varios  indicadores

```
#import Pkg; Pkg.add("IMFData")
using CSV, DataFrames, DataFramesMeta, IMFData

wd = @__DIR__
# cambiar por el directorio donde se quiere guardar los archivos, por ejemplo:
wd = "C:/Directorio_Trabajo/Julia/IMFData"

function get_df(x)
    try
        df = countries_indicators[x].series
        df.country .= countries_indicators[x].area
        df.indicator .= countries_indicators[x].indicator
        df.frequency .= countries_indicators[x].frequency
        return df
    catch
        return DataFrames.DataFrame()
    end
end

ifs_structure  = IMFData.get_imf_datastructure("IFS")
ifs_indicators = ifs_structure["Parameter Values"]["CL_INDICATOR_IFS"]
countries  = ["GT","HN","SV","NI","CR","PA"]
#gdp_indicators = @where(
gdp_indicators = DataFramesMeta.@subset(
	ifs_indicators,
	DataFrames.occursin.("Gross Domestic Product", :description),
	DataFrames.occursin.("Domestic Currency", :description))
indicators = gdp_indicators[!,1]
countries_indicators = get_ifs_data(countries, indicators, "Q", 1900, 2100)
ndf = size(countries_indicators)[1]
df = map(_ -> DataFrames.DataFrame(), 1:ndf)
[df[i] = get_df(i) for i in 1:ndf]
df = vcat(df...)
CSV.write(
    wd * "/IMFData_query.csv",
    delim = ";",
    df)
df
```
