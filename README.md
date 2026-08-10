# Exoplanet Habitability Model

A one-class variational autoencoder for identifying potentially habitable exoplanets that do not resemble Earth.

Traditional Earth-similarity indices can penalize planets that differ substantially from Earth. This project learns the characteristics of ordinary exoplanets and identifies planets that the model reconstructs poorly. Some of these anomalies may be physically plausible, non-Earth-like habitability candidates.

## Project Contents

- `pipeline.ipynb`: Complete Colab notebook containing data download, cleaning, baseline metric calculations, VAE training, anomaly scoring, physical filtering, and visualizations.
- `full_ranking.csv`: Rankings and scores for all 5,111 cleaned planets.
- `clean_anomaly_ranking.csv`: Physically filtered anomaly ranking.
- `vae_model.pth`: Trained PyTorch model weights.
- `vae_vs_baselines.png`: Scatter plots comparing VAE anomaly scores with ESI and CDHS.
- `README.md`: Project documentation.

## Data

The data come from the [NASA Exoplanet Archive](https://exoplanetarchive.ipac.caltech.edu/), using the `pscomppars` table.

The model uses five parameters with reliable measurements:

| Column | Description | Units |
|---|---|---|
| `pl_rade` | Planet radius | Earth radii |
| `pl_dens` | Planet bulk density | Earth densities |
| `pl_insol` | Stellar insolation received by the planet | Earth = 1 |
| `st_teff` | Stellar effective temperature | Kelvin |
| `pl_orbeccen` | Orbital eccentricity | Dimensionless |

After removing missing and obviously incorrect values, the dataset contains 5,111 planets.

> This project uses bulk planetary and stellar observables only. It does not include atmospheric composition, atmospheric pressure, surface chemistry, or biological data.

## Baseline Habitability Scores

The project compares the VAE anomaly score with three habitability-related metrics:

- **Earth Similarity Index (ESI):** Calculated using both equal weights and weights inspired by Schulze-Makuch et al. (2011).
- **Cobb-Douglas Habitability Score (CDHS):** Uses elasticity coefficients optimized to assign higher scores to Earth-like planets and lower scores to clearly non-habitable reference planets.
- **Simplified SEPHI:** Combines an atmospheric-retention probability with a habitable-zone membership function.

The **Planetary Habitability Index (PHI)** is not calculated because it requires atmospheric chemistry and solvent information that is unavailable for most exoplanets.

All calculated scores are included in `full_ranking.csv`.

## VAE Model

### Architecture

- Input layer: 5 features
- Encoder: `5 → 16 → 8`
- Latent space: 2 dimensions using a mean and log-variance
- Decoder: `2 → 8 → 16 → 5`
- Loss function: Reconstruction mean squared error plus KL divergence
- Optimizer: Adam
- Learning rate: `1e-3`
- Training duration: 200 epochs

### Training Data

The VAE is trained without habitability labels.

The training set contains planets that are outside both of the following broad ranges:

- Insolation: `0.35–2.0` Earth fluxes
- Radius: `0.5–2.5` Earth radii

This produces 5,037 training samples representing ordinary planets outside the selected candidate region.

### Anomaly Score

Each planet receives a combined anomaly score based on two measurements:

1. **Reconstruction error:** Mean squared error between the original five-dimensional input and the VAE reconstruction.
2. **Latent Mahalanobis distance:** Distance between the planet's latent mean and the training-set centroid, calculated using the training covariance matrix.

Both values are min-max normalized and added together to produce the final `VAE_anomaly` score.

Higher values indicate that a planet is more unusual relative to the training distribution.

## Physical Filtering

The highest raw anomaly scores included physically unrealistic entries, such as extremely high densities and very large planetary radii.

The cleaned ranking removes planets that meet either of these conditions:

- `pl_dens < 0.1` or `pl_dens > 10` Earth densities
- `pl_rade > 20` Earth radii

The filtered results are available in `clean_anomaly_ranking.csv`.

## How to Run

### Google Colab

Google Colab is the recommended way to run the project.

1. Upload `pipeline.ipynb` to Google Colab.
2. Run the notebook cells in order.
3. The notebook downloads data from the NASA Exoplanet Archive.
4. If the automatic download fails, use the fallback link provided in the notebook or upload a file named `exoplanets_clean.csv`.
5. Run all remaining cells.
6. Download the generated rankings, model weights, and plots from the Colab file sidebar.

Approximate training time:

- GPU runtime: 5 minutes
- CPU runtime: 10 minutes

### Local Installation

The project requires Python 3.8 or newer.

Install the dependencies with:

```bash
pip install pandas numpy torch scikit-learn scipy matplotlib astroquery
```

Then open `pipeline.ipynb` and run the notebook cells in order. The same fallback data-loading process is used if the NASA Exoplanet Archive download is unavailable.

## Results

After physical filtering, the highest-ranked anomalies include objects such as **K2-18 b** and several warm super-Earths with eccentric orbits.

Some of these planets have ESI values below 0.5 because their radius or insolation differs from Earth's. However, they may still fall within the selected habitable-insolation range and have densities consistent with potentially water-rich interiors.

The VAE identifies unusual planets that Earth-similarity metrics may overlook. However, the model is not a habitability detector. It is an unsupervised screening tool for prioritizing planets for further investigation.

## Limitations

- The model uses only five bulk observables.
- Atmospheric composition and surface conditions are not included.
- No direct evidence of liquid water or life is inferred.
- The training set is defined using manually selected radius and insolation ranges.
- Anomaly scores indicate statistical unusualness, not confirmed habitability.
- Results depend on data quality, preprocessing, normalization, model architecture, and filtering choices.
- A high anomaly score does not necessarily indicate that a planet is habitable.

## Authors

- Afeefa Anwar
- Yana Jain
- Erica Poku
- Eric Chen

## Citation

If you use this code, model, or rankings, please cite the associated paper when it becomes available or link to this repository.

Citation details coming soon.
```
