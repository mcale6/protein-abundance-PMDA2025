# protein-abundance-PMDA2025
- Clone the repo to get the training file all_organisms_filtered_without_M.musculus_KIDNEY.parquet
- Create an env with this packages: !pip install pandas numpy (pyarrow)
- Check with the code below if everything is okay with the data and you can load it.
- Store the files in a way most comfortable for you.
- Test data comign soon.

```python
import pandas as pd
import numpy as np 
#import pyarrow.parquet as pq

file_path = '/protein-abundance-PMDA2025/all_organisms_filtered_without_M.musculus_KIDNEY.parquet'
#schema = pq.read_schema(file_path)
#print(schema.names)
df = pd.read_parquet(file_path)

df['integrated_score'] = np.where(df['is_integrated'] == True, df['quality_score'], np.nan)
df['non_integrated_score'] = np.where(df['is_integrated'] == False, df['quality_score'], np.nan)

species_breakdown_updated = df.groupby('organism_name').agg(
    total_protein_entries=('EnsemblProteinID', 'count'),
    unique_proteins_name=('UniprotEntryName', 'nunique'),
    unique_proteins_id=('UniprotAccession', 'nunique'),
    avg_quality_score=('quality_score', 'mean'),
    avg_integrated_quality=('integrated_score', 'mean'),
    avg_non_integrated_quality=('non_integrated_score', 'mean')
)

species_breakdown_updated = species_breakdown_updated.sort_values(by='total_protein_entries', ascending=False)
score_cols = ['avg_quality_score', 'avg_integrated_quality', 'avg_non_integrated_quality']
species_breakdown_updated[score_cols] = species_breakdown_updated[score_cols].round(2)

print(species_breakdown_updated.head(5))
```
| organism_name                     |   total_protein_entries |   unique_proteins_name |   unique_proteins_id |   avg_quality_score |   avg_integrated_quality |   avg_non_integrated_quality |
|:----------------------------------|------------------------:|-----------------------:|---------------------:|--------------------:|-------------------------:|-----------------------------:|
| H.sapiens                         |             1.79753e+06 |                  19047 |                18673 |               19.97 |                    29.83 |                        17.46 |
| M.musculus                        |        489864           |                  19439 |                19117 |               17.39 |                    25.4  |                        15.18 |
| A.thaliana                        |        426081           |                  21866 |                21763 |               11.64 |                    16.54 |                         9.62 |
| Taestivum                         |        247308           |                  38372 |                38372 |               17.27 |                    22.55 |                        13.58 |