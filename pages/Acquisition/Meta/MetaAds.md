---
title: Meta Ads
---

<script>
    let myColors = [
'#B9E8FF','#5572FF','#D7E4FC','#FFE770','#FFF3B8', '#72D1FF','#B9E8FF','#f1f78e','#AAB9FF','#f1f78e','#5572FF','#9b81f6','#D7E4FC','#FFE770','#D7E4FC', '#72D1FF','#B9E8FF','#5572FF','#AAB9FF','#f1f78e'
    ]
</script>

<DateRange
    name=date
    start=2022-05-01
    end=2024-12-31
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

### Évolution du budget dépensé

<BigValue 
  data={big_meta} 
  value=spend
  title="Budget dépensé"
  fmt=eur0
/>

<BarChart 
    data={meta} 
    x=date 
    y=total_spend
    fmt=eur0
    sort=date
    colorPalette={myColors}
/>

### Répartition du budget dépensé entre acquisition et retargeting

```sql big_meta_acq
select sum(total_spend_facebook) as spend
from bigquery.meta_ads
where date between '${inputs.date.start}' and '${inputs.date.end}' and campaign_type ='ACQ'
```
```sql big_meta_rtg
select sum(total_spend_facebook) as spend
from bigquery.meta_ads
where date between '${inputs.date.start}' and '${inputs.date.end}' and campaign_type ='RTG'
```
<BigValue 
  data={big_meta_acq} 
  value=spend
  title="Budget ACQ"
  fmt=eur0
/>
<BigValue 
  data={big_meta_rtg} 
  value=spend
  title="Budget RTG"
  fmt=eur0
/>

<BarChart 
    data={meta} 
    x=date 
    y=total_spend
    fmt=eur0
    sort=date
    colorPalette={myColors}
    series=campaign_type
/>

### Part du budget dépensé entre acquisition et retargeting

```sql big_meta_rtg_100
SELECT
    (SELECT SUM(total_spend_facebook) AS spend_acq
     FROM bigquery.meta_ads
     WHERE date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}'
       AND campaign_type = 'RTG') / 
    (SELECT SUM(total_spend_facebook) AS all_spend
     FROM bigquery.meta_ads
     WHERE date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}') AS percentage_spend_acq
```
```sql big_meta_acq_100
SELECT
    (SELECT SUM(total_spend_facebook) AS spend_acq
     FROM bigquery.meta_ads
     WHERE date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}'
       AND campaign_type = 'ACQ') / 
    (SELECT SUM(total_spend_facebook) AS all_spend
     FROM bigquery.meta_ads
     WHERE date BETWEEN '${inputs.date.start}' AND '${inputs.date.end}') AS percentage_spend_acq
```
<BigValue 
  data={big_meta_acq_100} 
  value=percentage_spend_acq
  title="Budget ACQ"
  fmt=pct0
/>
<BigValue 
  data={big_meta_rtg_100} 
  value=percentage_spend_acq
  title="Budget RTG"
  fmt=pct0
/>

<BarChart 
    data={meta} 
    x=date 
    y=total_spend
    fmt=eur0
    sort=date
    colorPalette={myColors}
    series=campaign_type
    type=stacked100
/>