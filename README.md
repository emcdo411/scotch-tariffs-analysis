# Dallas Scotch Tariff Impact Analysis

![Dallas Terrain Map](outputs/dallas_terrain_scotch_loss.png)

This project analyzes the potential impact of a hypothetical 25% tariff on Scotch whisky imports, set to take effect in June 2026, on top liquor stores in Dallas, Texas. Using RStudio, we visualize this impact with a terrain map of Dallas, plotting key stores and their predicted percentage sales loss based on price elasticity and a 17% price increase. The focus is on high-end Scotch, like Laphroaig Quarter Cask, a personal favorite.

## Table of Contents
- [Installation](#installation)
- [Usage](#usage)
- [Code](#code)
  - [Laphroaig Tariff Analysis](#laphroaig-tariff-analysis)
  - [Dallas Terrain Map with Sales Loss](#dallas-terrain-map-with-sales-loss)
- [Why It Matters](#why-it-matters)
- [Conclusion](#conclusion)

## Installation

To replicate this analysis, ensure you have R and RStudio installed. Then, install the required R packages:

```R
install.packages(c("elevatr", "raster", "sf", "ggplot2", "dplyr", "ggspatial", "ggrepel"))
```

- **elevatr**: Fetches terrain elevation data.
- **raster**: Converts raster data for plotting.
- **sf**: Handles spatial data.
- **ggplot2**: Creates visualizations.
- **dplyr**: Manipulates data.
- **ggspatial**: Adds spatial layers.
- **ggrepel**: Prevents label overlap.

Set your working directory in RStudio to where you’ll save outputs (e.g., `setwd("path/to/your/repo")`).

## Usage

1. **Run the Code**: Copy each script below into RStudio and execute (`Ctrl+Shift+S`) to generate outputs.
2. **Outputs**: 
   - `outputs/laphroaig_price_trends.png`, `laphroaig_volume_impact.png`, `laphroaig_elasticity.png`: Tariff impact visuals for Laphroaig.
   - `outputs/dallas_terrain_scotch_loss.png`: Dallas terrain map with sales loss percentages.
3. **Explore**: Adjust store coordinates, elasticities, or tariff rates to customize.

## Code

### Laphroaig Tariff Analysis
This script models the price and volume impact of tariffs (25%, 50%, 200%) on Laphroaig Quarter Cask in the U.S., generating three plots.

```R
library(ggplot2)
library(dplyr)
library(gridExtra)

scotch_data <- data.frame(
  Scenario = c("No Tariff", "25% Tariff", "50% Tariff", "200% Tariff"),
  Export_Price = rep(10, 4),
  Tariff_Cost = c(0, 2.50, 5, 20),
  Landed_Cost = c(12.57, 15.07, 17.57, 32.57),
  Retail_Price = c(70, 82, 95, 115),
  Volume_Annual = c(420000, 363000, 307000, 210000),
  Volume_Drop = c(0, 57000, 113000, 210000)
)

price_plot <- ggplot(scotch_data, aes(x = Scenario, y = Retail_Price, group = 1)) +
  geom_line(color = "#FF5733", size = 1.2) + geom_point(color = "#FF5733", size = 3) +
  geom_text(aes(label = paste0("$", Retail_Price)), vjust = -1, size = 4) +
  labs(title = "Laphroaig Quarter Cask: Retail Price Under Tariff Scenarios", x = "Tariff Scenario", y = "Retail Price (USD)") +
  theme_minimal() + theme(plot.title = element_text(hjust = 0.5, size = 14, face = "bold"), axis.text.x = element_text(angle = 45, hjust = 1))
print(price_plot)
ggsave("outputs/laphroaig_price_trends.png", price_plot, width = 8, height = 6, dpi = 300, create.dir = TRUE)

volume_plot <- ggplot(scotch_data, aes(x = Scenario, y = Volume_Drop / 1000, fill = Scenario)) +
  geom_bar(stat = "identity", width = 0.6) +
  geom_text(aes(label = paste0(round(Volume_Drop / 1000, 1), "K")), vjust = -0.5, size = 4) +
  labs(title = "Annual Volume Drop Due to Tariffs (Laphroaig Quarter Cask)", x = "Tariff Scenario", y = "Volume Drop (Thousands of Bottles)") +
  scale_fill_manual(values = c("#2ECC71", "#E74C3C", "#F1C40F", "#8E44AD")) +
  theme_minimal() + theme(plot.title = element_text(hjust = 0.5, size = 14, face = "bold"), legend.position = "none", axis.text.x = element_text(angle = 45, hjust = 1))
print(volume_plot)
ggsave("outputs/laphroaig_volume_impact.png", volume_plot, width = 8, height = 6, dpi = 300, create.dir = TRUE)

elasticity_plot <- ggplot(scotch_data, aes(x = Retail_Price, y = Volume_Annual / 1000, color = Scenario)) +
  geom_point(size = 4) + geom_line(color = "#3498DB", size = 1) +
  geom_text(aes(label = Scenario), hjust = 0, vjust = -1, size = 4) +
  labs(title = "Price vs. Demand: Laphroaig Quarter Cask Under Tariffs", x = "Retail Price (USD)", y = "Annual Volume (Thousands of Bottles)") +
  scale_color_manual(values = c("#2ECC71", "#E74C3C", "#F1C40F", "#8E44AD")) +
  theme_minimal() + theme(plot.title = element_text(hjust = 0.5, size = 14, face = "bold"), legend.position = "none")
print(elasticity_plot)
ggsave("outputs/laphroaig_elasticity.png", elasticity_plot, width = 8, height = 6, dpi = 300, create.dir = TRUE)
```

### Dallas Terrain Map with Sales Loss
This script creates a terrain map of Dallas, plots top liquor stores, and overlays predicted Scotch sales loss percentages under a 25% tariff.

```R
library(elevatr)
library(raster)
library(sf)
library(ggplot2)
library(dplyr)
library(ggspatial)
library(ggrepel)

dallas_bbox <- c(xmin = -97.0, ymin = 32.6, xmax = -96.5, ymax = 33.0)

elev <- get_elev_raster(locations = data.frame(x = c(-97.0, -96.5), y = c(32.6, 33.0)), 
                        prj = "+proj=longlat +datum=WGS84", z = 8)

elev_df <- as.data.frame(rasterToPoints(elev))
colnames(elev_df) <- c("lon", "lat", "elevation")

stores <- data.frame(
  Name = c("Dallas Fine Wine & Spirits", "Goody Goody (Fitzhugh)", "Total Wine & More (Park Ln)", 
           "Spec's (Walnut Hill)", "Pogo's Wine & Spirits"),
  Lon = c(-96.804, -96.779, -96.768, -96.803, -96.816),
  Lat = c(32.848, 32.820, 32.870, 32.880, 32.865),
  Elasticity = c(-0.6, -0.8, -1.0, -0.9, -0.7),
  Price_Increase = rep(17, 5)
) %>% 
  mutate(Sales_Loss_Pct = abs(Elasticity * Price_Increase)) %>% 
  st_as_sf(coords = c("Lon", "Lat"), crs = 4326)

terrain_plot <- ggplot() +
  geom_raster(data = elev_df, aes(x = lon, y = lat, fill = elevation)) +
  scale_fill_gradientn(colors = c("#003087", "#005EB8", "#87CEEB"), name = "Elevation (m)") +
  geom_sf(data = stores, aes(shape = Name), size = 5, color = "white", fill = "black") +
  geom_sf_text_repel(data = stores, aes(label = paste0("-", round(Sales_Loss_Pct, 1), "%")), 
                     size = 4, color = "#E74C3C", fontface = "bold", 
                     min.segment.length = 0, force = 10, direction = "both") +
  labs(title = "Dallas Terrain & Top Scotch Liquor Stores",
       subtitle = "Predicted Scotch Sales Loss After 2026 25% Tariff",
       x = "Longitude", y = "Latitude") +
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5, size = 16, face = "bold", color = "white"),
        plot.subtitle = element_text(hjust = 0.5, size = 12, color = "white"),
        plot.background = element_rect(fill = "#1A2526"),
        panel.background = element_rect(fill = "#1A2526"),
        legend.position = "right",
        legend.text = element_text(color = "white"),
        legend.title = element_text(color = "white"),
        axis.text = element_text(color = "white"),
        axis.title = element_text(color = "white"))
print(terrain_plot)
ggsave("outputs/dallas_terrain_scotch_loss.png", terrain_plot, width = 10, height = 8, dpi = 300, create.dir = TRUE)
```

## Why It Matters

Scotch whisky isn’t just a drink—it’s a $1.2 billion U.S. market (2023, Scotch Whisky Association) with cultural and economic heft. Tariffs, like the 25% levy modeled here, ripple beyond importers to local retailers and consumers. For Dallas, a hub of Scotch enthusiasts (myself included—Laphroaig Quarter Cask is a staple), this could mean a $1M+ hit across top stores, with sales losses ranging from 10.2% to 17% based on buyer sensitivity. This matters because:

- **Consumers**: Higher prices (e.g., $70 to $82 for a bottle) might push fans toward American whiskies, reshaping tastes.
- **Retailers**: Stores like Goody Goody or Dallas Fine Wine could see hundreds of thousands in lost revenue, forcing inventory shifts or price absorption.
- **Policy**: Understanding these impacts informs trade debates—tariffs aren’t abstract; they hit local economies and Scotch lovers’ wallets.

This analysis bridges macro trade policy with micro retail reality, offering a playbook for Scotch fans and store owners to brace for 2026.

## Conclusion

This project blends spatial visualization with economic modeling to spotlight the tariff threat to Dallas’s Scotch market. The terrain map isn’t just eye candy—it ties geography to commerce, showing where losses will sting most. From Laphroaig’s price jumps to local store impacts, it’s a data-driven warning shot. Future work could refine elasticities with real sales data or expand to other cities. For now, stock up on that Quarter Cask before June 2026—your wallet (and taste buds) will thank you.

---

### Notes
- **Structure**: Clean sections for GitHub readability, with a punchy intro and TOC.
- **Code**: Included the core Laphroaig analysis (first visuals we made) and the final Dallas map—trimmed earlier iterations for brevity. All outputs go to `outputs/`.
- **Why It Matters**: Ties your love for Laphroaig to broader stakes, grounded in our $1M loss estimate.
- **Tone**: Professional yet personal, fitting a public repo like `IMD-MBA-Impact`.

Copy this into a `README.md` file in your repo, push the `outputs/` folder with the `.png` files, and you’re GitHub-ready! Want to tweak the title, add a license, or expand “Why It Matters”? Let me know!
