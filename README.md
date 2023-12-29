# factbook

A little json parser and website for the CIA World Factbook found at https://github.com/factbook/factbook.json.

## Using JS

```js
function fetchData() {

var random = Math.floor(Math.random() * files.length);
var file = files[random];
var jsonFileUrl = 'https://raw.githubusercontent.com/factbook/factbook.json/master/'+file;
var countryCode = file.substring(file.indexOf('/') + 1, file.indexOf('.'));
var countryName = Object.keys(codes).find(key => codes[key] === countryCode.toUpperCase()) || countryCode.toUpperCase();

return fetch(jsonFileUrl)
.then(response => response.json())
.then(jsonObject => {

// random key
function getAllKeys(obj) {
let keys = [];
for (let key in obj) {
if (typeof obj[key] === 'object') {
keys = keys.concat(getAllKeys(obj[key]).map(subKey => `${key}.${subKey}`));
} else {
keys.push(key);
}
}
return keys;
}

// find matching keys
function getKeysAndValuesWithSameFirstTwoParts(obj, selectedKey) {
// Split the selected key by '.'
const keyParts = selectedKey.split('.');

// Extract the first and second parts
const [firstPart, secondPart] = keyParts;

// Find all keys and values with the same first and second parts
const matchingKeysAndValues = getAllKeys(obj)
.filter(key => {
const parts = key.split('.');
return parts.length >= 2 && parts[0] === firstPart && parts[1] === secondPart;
})
.map(key => ({
key,
value: key.split('.').reduce((acc, key) => acc[key], obj)
}));

return matchingKeysAndValues;
}

// Get all keys of the JSON object, including nested keys
const allKeys = getAllKeys(jsonObject);
// Define a wildcard pattern to exclude specific keys (replace with your pattern)
const excludeWildcard = /note/;
// Get all keys of the JSON object, excluding keys with the specified wildcard
const filteredKeys = getAllKeys(jsonObject, excludeWildcard);
// Select a random key
const randomKey = filteredKeys[Math.floor(Math.random() * filteredKeys.length)];
// find matching keys
const matchingKeysAndValues = getKeysAndValuesWithSameFirstTwoParts(jsonObject, randomKey);
// Split the random key by '.'
const keyParts = randomKey.split('.');

// start div
const div = document.createElement('div');
resultContainer.appendChild(div);
div.classList.add(`flex-column`);

// Add title
contentToAppend = `<div class="part-1" style="margin:5px">${counterValue}.</div><div class="part-2">${keyParts[0]}</div><div class="part-1">${keyParts[1]} ~ ${countryName}</div>`;
div.innerHTML += contentToAppend;

counterValue += 1;

// Add content
matchingKeysAndValues.forEach((item) => {
const itemParts = item.key.split('.');
const remainingParts = itemParts.slice(2).join('.');
if (remainingParts !== 'text') {
contentToAppend = `<p style="font-size:12px; font-weight:600; margin-top:5px;">${remainingParts.replace(/text/g, '').replace(/\./g, '')}</p><p>${item.value.replace(/<strong>note:\s*<\/strong>/g, '').replace(/<strong>note\s*<\/strong>:/g, '')}</p>`;
div.innerHTML += contentToAppend;
} else {
contentToAppend = `<p>${item.value.replace(/<strong>note:\s*<\/strong>/g, '').replace(/<strong>note\s*<\/strong>:/g, '')}</p>`;
div.innerHTML += contentToAppend;
}
});
div.innerHTML += `<hr>`;

})} <!-- end fetch function -->

const numberOfFetches = 10;
for (let i = 0; i < numberOfFetches; i++) {
fetchData();
}
```

## Using psql

Scraping  
```bash
for region in africa antarctica australia-oceania central-america-n-caribbean central-asia east-n-southeast-asia europe meta middle-east north-america oceans south-america south-asia world; do
  curl https://github.com/factbook/factbook.json/tree/master/${region} | sed -e 's/{/\n{/g' | grep '"contentType":"file"' | grep '.json' | grep -v 'package.json' | grep -v 'categories' | sed -e 's/^.*path":"/https:\/\/raw\.githubusercontent\.com\/factbook\/factbook\.json\/master\//g' -e 's/".*$//g' | while read url; do
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
