# factbook

So far, a little json parser and website for the CIA World Factbook found at https://github.com/factbook/factbook.json.

## Processing in psql

Scraping  
```bash
for region in africa antarctica australia-oceania central-america-n-caribbean central-asia east-n-southeast-asia europe meta middle-east north-america oceans south-america south-asia world; do
  curl https://github.com/factbook/factbook.json/tree/master/${region} | sed -e 's/{/\n{/g' | grep '"contentType":"file"' | grep '.json' | grep -v 'package.json' | grep -v 'categories' | sed -e 's/^.*path":"/https:\/\/github\.com\/factbook\/factbook\.json\/tree\/master\//g' -e 's/".*$//g' | while read url; do
    wget ${url}; sleep $[ ( $RANDOM % 10 )  + 1 ]s
    echo $url
  done
done
```

Importing  
```bash
echo -e 'code\tbackground\tlocation\tarea_total\tarea_land\tarea_water\tarea_comparative\tclimate\tterrain\tnatural_resources\tlanduse_agricultural\tlanduse_forest\tlanduse_other\tpop_distribution\tnatural_hazards\tnote\tpop\tlanguage\tage_structure\tmedian_age\tpop_growth_rate\tbirth_rate\tdeath_rate\tnet_migration_rate\turban_pop\turbanization_rate\teconomic_overview\tgdp\tgdp_growth_rate\tgdp_per_capita\tgdp_by_sector\tgdp_end_use\tagricultural_products\tindustries\texports\texport_partners\texport_commodities\timports\timport_partners\timport_commodities' > factbook.csv

ls *.json | while read file; do   jq -j 'input_filename, "\t", ."Introduction"."Background".text, "\t", ."Geography"."Location".text, "\t", ."Geography"."Area"."total".text, "\t", ."Geography"."Area"."land".text, "\t", ."Geography"."Area"."water".text, "\t", ."Geography"."Area - comparative".text, "\t", ."Geography"."Climate".text, "\t", ."Geography"."Terrain".text, "\t", ."Geography"."Natural resources".text, "\t", ."Geography"."Land use"."agricultural land".text, "\t", ."Geography"."Land use"."forest".text, "\t", ."Geography"."Land use"."other".text, "\t", ."Geography"."Population distribution".text, "\t", ."Geography"."Natural hazards".text, "\t", ."Geography"."Geography - note".text, "\t", ."People and Society"."Population".text, "\t", ."People and Society"."Languages"."Languages".text, "\t", ."People and Society"."Age structure", "\t", ."People and Society"."Median age"."total".text, "\t", ."People and Society"."Population growth rate".text, "\t", ."People and Society"."Birth rate".text, "\t", ."People and Society"."Death rate".text, "\t", ."People and Society"."Net migration rate".text, "\t", ."People and Society"."Urbanization"."urban population".text, "\t", ."People and Society"."Urbanization"."rate of urbanization".text, "\t", ."Economy"."Economic overview".text, "\t", ."Economy"."Real GDP (purchasing power parity)", "\t", ."Economy"."Real GDP growth rate", "\t", ."Economy"."Real GDP per capita", "\t", ."Economy"."GDP - composition, by sector of origin", "\t", ."Economy"."GDP - composition, by end use", "\t", ."Economy"."Agricultural products".text, "\t", ."Economy"."Industries".text, "\t", ."Economy"."Exports", "\t", ."Economy"."Exports - partners".text, "\t", ."Economy"."Exports - commodities".text, "\t", ."Economy"."Imports", "\t", ."Economy"."Imports - partners".text, "\t", ."Economy"."Imports - commodities".text' ${file} | tr -d '\n' | tr -s ' ' | sed -e 's/\.json//g' -e 's/"//g' >> factbook.csv; echo -e '\n' >> factbook.csv; done
sed -i '/^[[:blank:]]*$/ d' factbook.csv

# import psql
psql -d world -c "CREATE TABLE factbook (code text, background text, location text, area_total text, area_land text, area_water text, area_comparative text, climate text, terrain text, natural_resources text, landuse_agricultural text, landuse_forest text, landuse_other text, pop_distribution text, natural_hazards text, note text, pop text, language text, age_structure text, median_age text, pop_growth_rate text, birth_rate text, death_rate text, net_migration_rate text, urban_pop text, urbanization_rate text, economic_overview text, gdp text, gdp_growth_rate text, gdp_per_capita text, gdp_by_sector text, gdp_end_use text, agricultural_products text, industries text, exports text, export_partners text, export_commodities text, imports text, import_partners text, import_commodities text);"
psql -d world -c "\COPY factbook FROM 'factbook.csv' WITH DELIMITER E'\t' CSV HEADER;"
psql -d world -c "UPDATE factbook SET code = UPPER(code);"
```
