# Data-Handling-and-Visualization

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np

sns.set_theme(style="white", context="talk") 
plt.rcParams.update({
    'font.family': 'sans-serif',
    'axes.titleweight': 'bold',
    'axes.labelweight': 'bold',
    'axes.spines.top': False,
    'axes.spines.right': False
})

df = pd.read_csv('WHO_GHO.csv')
df = df.dropna(subset=['OBS_VALUE'])

anemia_label = 'Prevalence of anemia in pregnant women (aged 15-49)'
road_death_label = 'Road traffic crash deaths, age-standardized death rates (15+), per 100,000 population'


anemia_df = df[df['INDICATOR_LABEL'] == anemia_label].copy()
anemia_global = anemia_df.groupby('TIME_PERIOD')['OBS_VALUE'].agg(['mean', 'std', 'min', 'max']).reset_index()

road_2019 = df[(df['INDICATOR_LABEL'] == road_death_label) & (df['TIME_PERIOD'] == 2019)]
road_sex = road_2019[road_2019['SEX_LABEL'].isin(['Male', 'Female'])]


road_total = road_2019[road_2019['SEX_LABEL'] == 'Total'].sort_values('OBS_VALUE', ascending=False)
top_15_road = road_total.head(15)

anemia_df['Decade'] = (anemia_df['TIME_PERIOD'] // 10) * 10
anemia_summary = anemia_df.groupby('Decade')['OBS_VALUE'].agg(['count', 'mean', 'std', 'min', 'max']).round(2)
anemia_summary.columns = ['Count', 'Mean (%)', 'Std Dev', 'Min (%)', 'Max (%)']
anemia_summary.to_csv('anemia_summary_table.csv')


plt.figure(figsize=(10, 6))
plt.plot(anemia_global['TIME_PERIOD'], anemia_global['mean'], marker='s', markersize=8, color='#2c7fb8', label='Global Average')
plt.fill_between(anemia_global['TIME_PERIOD'], anemia_global['min'], anemia_global['max'], color='#2c7fb8', alpha=0.1, label='Range (Min to Max)')
plt.title('Progress in Global Nutrition: Prevalence of Anemia (2000-2019)', pad=20)
plt.xlabel('Year')
plt.ylabel('Prevalence (%)')
plt.xticks(np.arange(2000, 2020, 2))
plt.legend(frameon=False, loc='upper right')
plt.tight_layout()
plt.savefig('figure1_anemia_trends.png', dpi=300)

plt.figure(figsize=(10, 6))
sns.kdeplot(data=road_sex, x='OBS_VALUE', hue='SEX_LABEL', fill=True, common_norm=False, palette=['#e66101', '#5e3c99'], alpha=.5, linewidth=2)
plt.title('The Gender Divide in Road Safety: Density of Death Rates (2019)', pad=20)
plt.xlabel('Age-standardized death rate (per 100,000 population)')
plt.ylabel('Density of Countries')
plt.tight_layout()
plt.savefig('figure2_road_gender_density.png', dpi=300)

plt.figure(figsize=(10, 8))
sns.barplot(data=top_15_road, x='OBS_VALUE', y='REF_AREA_LABEL', color='#d95f02')
plt.title('High-Risk Zones: Top 15 Countries by Road Fatality Rates (2019)', pad=20)
plt.xlabel('Deaths per 100,000 Population')
plt.ylabel('Country')

for i, v in enumerate(top_15_road['OBS_VALUE']):
    plt.text(v + 0.5, i + .15, str(round(v, 1)), color='black', fontweight='bold', size=10)
plt.tight_layout()
plt.savefig('figure3_road_ranking.png', dpi=300)

print("Visualizations and Table generated successfully.")
