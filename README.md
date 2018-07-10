# Analysis

## Overview
This section describes market basket analysis using the data from a UK-based online retailer to discover the co-occurrence relationships among customers’ purchase activities, such as the likelihood to purchase a candle holder if the customer already has candles and matches in their shopping cart. Such an analysis is commonly used in marketing to increase the chance of cross-selling, provide recommendations to customers (e.g., based on their browsing history) and deliver targeted marketing (e.g., offer coupons to customers for products that are frequently purchased together with items that the customers recently bought).

Initial exploration and cleaning of the data are described in the [previous section](https://eagronin.github.io/market-basket-prepare/).

The results are reported in the [next section](https://eagronin.github.io/market-basket-report/).

The analysis for this project was performed in R.  For an introduction to working with the arules package in R please refer to this [tutorial](http://www.learnbymarketing.com/1043/working-with-arules-transactions-and-read-transactions/).

## Finding Associations Across Consumer Purchases

The only features in the dataset that we need to perform market basket analysis are the ID, which represents a unique ID for a transaction, and Description, which represents item description.  In order to prepare the data for market basket analsis in R, we need to convert the `items` dataset, which is currently in dataframe format into transactions format.  To perform this conversion we first need to reshape our dataframe into a specific format that can be converted into transactions format.  

The following code removes the features that are no longer needed from the dataset, and reshapes the dataset into a "wide" format, i.e., we will create a matrix with each row corresponding to a unique ID and each column corresponding to a unique Description.  The cells in the matrix are going to take the values of either TRUE, when an item with the description as identified by its column name was purchased by the customer and on the date as specified by the corresponding row, or FALSE otherwise:

```R
keeps = c("ID", "Description")
temp = data.frame(ID = as.factor(items$ID), Description = as.factor(items$Description), const = TRUE)
wide = reshape(data = temp, idvar = "ID", timevar = "Description", direction = "wide")
wide = as.matrix(wide[,-1])          # drop the transaction ID as it is no longer used
wide[is.na(wide)] = FALSE.           # clean up the missing values to be FALSE
colnames(wide) = gsub(x=colnames(wide), pattern="const\\.", replacement="")    # clean up column names
```

After executing the above code, we are ready to convert the dataset into transaction fromat.  The code below performs this conversion and outputs a frequency plot showing the number of purchases of the top 10 items.  The plot is shown in the [next section](https://eagronin.github.io/market-basket-report/). 

```R
library(arules)
trans = as(wide, "transactions")
itemFrequencyPlot(trans, topN=10, type='absolute'
                  , ylab = "Item Frequency (Absolute)", main = "Figure 3. Number of Purchases of the Top 10 Items"
                  , col = "blue", cex.axis = 0.8, cex.main=1.5, cex.lab = .8, cex.names = 0.8)
```

Finally, we use the arules package to run the apriori algorithm for the minimum support of 0.005 and minimum confidence of 0.95, and output the results:

```R
rules = apriori(trans, parameter = list(supp=0.005, conf=0.95))
rules = sort(rules, by = 'support', decreasing = TRUE)
summary(rules)
inspect(rules[1:10])
plot(rules, main = "Figure 4. Scatter Plot of 354 Rules")
```

Next step: [Results](https://eagronin.github.io/market-basket-report/)
