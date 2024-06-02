---
title: Google Ads
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


# Google Ads

```sql google
select *
from bigquery.google_ads
where date between '${inputs.date.start}' and '${inputs.date.end}' 
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
    x=date 
    y=total_spend
    fmt=eur0
    sort=date
    colorPalette={myColors}
/>
