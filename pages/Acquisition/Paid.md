---
title: Paid Acquisition
---

<script>
    let myColors = [
'#B9E8FF','#5572FF','#D7E4FC','#FFE770','#FFF3B8', '#72D1FF','#B9E8FF','#f1f78e','#AAB9FF','#f1f78e','#5572FF','#9b81f6','#D7E4FC','#FFE770','#D7E4FC', '#72D1FF','#B9E8FF','#5572FF','#AAB9FF','#f1f78e'
    ]
</script>

```sql yesterday
SELECT CURRENT_DATE
```

<DateRange
    name=date
    start='2022-08-01'
    end='${yesterday}'
/>

<Dropdown
    name=d_time
    title="Période"
    defaultValue="date"
>
    <DropdownOption valueLabel="Jour" value="date" />
    <DropdownOption valueLabel="Mois" value="date_month" />
    <DropdownOption valueLabel="Trimestre" value="date_quarter" />
    <DropdownOption valueLabel="Année" value="date_year" />
</Dropdown>

# Overall


<BigValue 
  data={big_cac} 
  value=total_spend
  title="Marketing Spend"
  fmt=eur0
/>
<BigValue 
  data={big_new_customers} 
  value=new_customers
  title="Nouveaux Clients"
  fmt=num0
/>
<BigValue 
  data={big_cac} 
  value=global_cac
  title="Blended CAC"
  fmt=eur0
/>



```sql big_new_customers
SELECT sum(new_customers_count) as new_customers
FROM bigquery.marketing_costs
```

```sql big_cac
SELECT
    SUM(total_spend) AS total_spend,
    SUM(new_customers_count) AS total_new_customers,
    CASE
        WHEN SUM(new_customers_count) > 0 THEN SUM(total_spend) / SUM(new_customers_count)
        ELSE NULL
    END AS global_cac
FROM bigquery.marketing_costs
WHERE date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}'
```

```sql cac
SELECT
    ${inputs.d_time.value},
    SUM(total_spend) AS total_spend,
    SUM(new_customers_count) AS total_new_customers,
    CASE
        WHEN SUM(new_customers_count) > 0 THEN SUM(total_spend) / SUM(new_customers_count)
        ELSE NULL
    END AS global_cac
FROM bigquery.marketing_costs
WHERE date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}'
GROUP BY ${inputs.d_time.value}
```

```sql overall
SELECT
    ${inputs.d_time.value},
    sum(total_spend_facebook) AS spend,
    'Meta Ads' AS type
FROM
    bigquery.marketing_costs
WHERE
    date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}'
GROUP BY 
    ${inputs.d_time.value}

UNION ALL

SELECT
    ${inputs.d_time.value},
    sum(total_spend_google) AS spend,
    'Google Ads' AS type
FROM
    bigquery.marketing_costs
WHERE
    date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}'
GROUP BY 
    ${inputs.d_time.value}

```


<Dropdown
    name=metric
    title="Metric"
>
    <DropdownOption valueLabel="Spend" value="total_spend" />
    <DropdownOption valueLabel="Nouveaux clients" value="total_new_customers" />
</Dropdown>


<BarChart 
    data={cac} 
    x={inputs.d_time.value} 
    y={inputs.metric.value}
    y2=global_cac
    y2SeriesType=line
    y2Fmt=eur0
    sort={inputs.d_time.value}
    colorPalette={myColors}
    yAxisColor='#72D0FF'
/>

<BarChart 
    data={overall} 
    x={inputs.d_time.value} 
    y=spend
    fmt=eur0
    sort={inputs.d_time.value}
    colorPalette={myColors}
    series=type
/>



# Meta Ads

<Dropdown
    name=campaign_type
    title="Campaign type"
>
    <DropdownOption valueLabel="Tout" value="%" />
    <DropdownOption valueLabel="Retargeting" value="RTG" />
    <DropdownOption valueLabel="Acquisition" value="ACQ" />
</Dropdown>

```sql meta
select *
from bigquery.meta_ads
where date between '${inputs.date.start}' and '${inputs.date.end}' and campaign_type like '${inputs.campaign_type.value}' 
```

```sql big_meta
select sum(total_spend_facebook) as spend
from bigquery.meta_ads
where date between '${inputs.date.start}' and '${inputs.date.end}' and campaign_type like '${inputs.campaign_type.value}' 
```
<BigValue 
  data={big_meta} 
  value=spend
  title="Budget dépensé"
  fmt=eur0
/>

<BarChart 
    data={meta} 
    x={inputs.d_time.value}  
    y=total_spend_facebook
    fmt=eur0
    sort={inputs.d_time.value}  
    colorPalette={myColors}
/>

### [Analyses Meta Ads détaillées](/Acquisition/Meta/MetaAds/)

# Google Ads

```sql google
select *
from bigquery.google_ads
where date between '${inputs.date.start}' and '${inputs.date.end}' 
order by date asc
```

```sql big_google
select sum(total_spend_google) as spend
from bigquery.google_ads
where date between '${inputs.date.start}' and '${inputs.date.end}'
```
<BigValue 
  data={big_google} 
  value=spend
  title="Budget dépensé"
  fmt=eur0
/>

<BarChart 
    data={google} 
    x={inputs.d_time.value}   
    y=total_spend_google
    fmt=eur0
    sort={inputs.d_time.value}  
    colorPalette={myColors}
/>

### [Analyses Google Ads détaillées](/Acquisition/Google/GoogleAds/)

# ROI Calculator


Ce tableau est conçu pour aider à déterminer la combinaison optimale de dépenses marketing et de retour sur investissement (ROI) afin de maximiser la marge que vous pouvez réaliser chaque mois.

L'objectif est de trouver le juste équilibre entre le montant investi en marketing (représenté sur l'axe horizontal) et le ROI attendu (représenté sur l'axe vertical) pour obtenir la meilleure marge possible. En ajustant ces deux variables, vous pouvez prévoir les marges que ces dépenses engendreront et ainsi optimiser vos résultats.


<TextInput
    name=roi
    title="ROI"
    defaultValue=2
/>
<TextInput
    name=spend_start
    title="Marketing Spend Min."
    defaultValue=15000
/>
<TextInput
    name=spend_end
    title="Marketing Spend Max."
    defaultValue=30000
/>

```sql roi
SELECT
  cast(roi.generate_series / 10.0 as string) AS roi,
  cast(spend.generate_series as string) AS spend,
  (roi.generate_series / 10.0) * spend.generate_series as profit
FROM generate_series(${inputs.roi} * 10 - 5, ${inputs.roi} * 10 + 5, 1) AS roi
CROSS JOIN generate_series(${inputs.spend_start}, ${inputs.spend_end}, 1000) AS spend


```

<Heatmap 
    data={roi} 
    x=roi 
    y=spend 
    value=profit 
    valueFmt=num0
    xSort=roi 
    xSortOrder=asc
    ySort=spend 
    ySortOrder=desc
    colorPalette={['white','#50e3c2','#5516FD']}
/>


<Accordion>
  <AccordionItem title="Comment utiliser ce graphique">

    
Supposons que vous visiez un ROI de 2 et que vous prévoyez de dépenser au moins 30 000 euros en marketing. Sur le tableau, vous chercherez l'intersection de la ligne correspondant à un ROI de 2 et de la colonne pour un budget marketing de 30 000 euros. Disons que le point d'intersection indique une marge prévue de 70 000 euros. Cela signifierait que pour une dépense marketing de 30 000 euros avec un ROI de 2, vous pouvez vous attendre à générer une marge de 70 000 euros.

Vous pouvez également utiliser le tableau pour explorer différentes hypothèses. Par exemple, si vous augmentez vos dépenses marketing à 40 000 euros tout en visant un ROI de 2, observez quel sera l'impact sur votre marge. Ou bien, si vous souhaitez obtenir un ROI de 2,5 en maintenant vos dépenses marketing à 30 000 euros, voyez comment cela influence la marge escomptée.

Le tableau est doté d'une palette de couleurs permettant d'identifier visuellement les niveaux de performance : les teintes plus foncées peuvent signifier des marges plus importantes, vous permettant de repérer d'un coup d'œil les combinaisons les plus avantageuses pour votre stratégie marketing.

</AccordionItem>
</Accordion>