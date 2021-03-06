Regioes de Saude Brasil
================
Arthur Rios
04/06/2020

Esse repositório tem o objetivo de agrupar os arquivos de shapes dos
municípios de todo o Brasil com a informação da região de saúde.

O link para o csv está
[aqui](https://raw.githubusercontent.com/arzevedo/Regioes-de-Saude-Brasil/master/reg_saude_mun.csv)

Vamos utilizar o shape do @tbrugz do repositório
[geodata-br](https://github.com/tbrugz/geodata-br)

``` r
mun_shp <-
sf::read_sf("https://raw.githubusercontent.com/tbrugz/geodata-br/master/geojson/geojs-100-mun.json")
```

Agora vamos buscar as regiões de saúde por município Você pode encontrar
essa tabela no site do [Data
SUS](http://www2.datasus.gov.br/DATASUS/index.php?area=060206) e em
seguida clicar no link para as [bases
territoriais](ftp://ftp.datasus.gov.br/territorio/tabelas). Para fazer o
download do pdf com dicionário vá esse
[link](ftp://ftp.datasus.gov.br/territorio/doc/bases_territoriais.pdf).

``` r
reg_sau <- readr::read_csv("data/rl_municip_regsaud.csv")
```

Por algum motivo que eu imagino seja a indexação o código de município
da base de regiões de saúde tem o último dígito faltante. Vou cortar o
último dígito da base de shapes para compensar.

### Unindo as bases

``` r
mun_reg_sau <- mun_shp %>% 
  dplyr::mutate(
    mun_red = str_replace(id, ".{1}$", "")
  ) %>% 
  dplyr::left_join(reg_sau, by = c("mun_red" = "CO_MUNICIP"))

readr::write_csv(mun_reg_sau, path = "reg_saude_mun.csv")

readr::write_csv(mun_reg_sau %>% as.data.frame() %>% 
                   dplyr::select(-geometry), 
                 path = "reg_saude_raw.csv")
mun_reg_sau <- st_make_valid(mun_reg_sau)

dir.create("Reg_Saude_Shape")
sf::write_sf(mun_reg_sau, "Reg_Saude_Shape/reg_saude.shp")
```

## Um exemplo com a Bahia

Vamos observar uma comparação do output com a Bahia

``` r
mun_bahia <- mun_reg_sau %>% 
  dplyr::filter(startsWith(id, "29")) %>%  # Código Da BAHIA
  group_by(CO_REGSAUD) %>% 
  count()

mun_bahia %>% 
  ggplot(aes(fill = CO_REGSAUD)) +
  geom_sf(show.legend = FALSE,mapping = aes(geometry = geometry)) +
  theme_void()
```

![](README_files/figure-gfm/exemplo-1.png)<!-- -->

Comparando com a fonte oficial do governo da
[Bahia](http://www1.saude.ba.gov.br/mapa_bahia/indexch.asp):

![image](https://user-images.githubusercontent.com/36868624/83819649-cc40d300-a6a0-11ea-911e-536cc685965b.png)
