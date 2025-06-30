# Biom-File
Description how to open a BIOM file in R

#if (!require("BiocManager", quietly = TRUE))
  install.packages("BiocManager")

#BiocManager::install("rhdf5")

# Step 3: Load the rhdf5 library
library(rhdf5)

# Path to the .biom file
biom_file <- "B1.biom"

# Check file content
h5ls(biom_file)

# Read the OTU table
otu_data <- h5read(biom_file, "observation/matrix/data")
 print(otu_data)

 df <- read.csv("your_file.csv", row.names = 1)
 
 # Convert the row names to a column named 'ID'
 df <- df %>%
   rownames_to_column(var = "ID")
 
 # Extract only the required part of the ID (pattern: Gxxxx)
 df <- df %>%
   mutate(ID = sub(".*(G[0-9]+).*", "\\1", ID))  
 View(otu_data)
 
 # Read data components of the sparse matrix
 data <- h5read(biom_file, "/sample/matrix/data")  # Non-zero values
 indices <- h5read(biom_file, "/sample/matrix/indices")  # Row indices of non-zero values
 indptr <- h5read(biom_file, "/sample/matrix/indptr")  # Start indices for each column
 sample_ids <- h5read(biom_file, "/sample/ids")  # Column names (samples)
 observation_ids <- h5read(biom_file, "/observation/ids")  # Row names (taxa)
 
 # Number of rows (taxa) and columns (samples)
 num_rows <- length(observation_ids)
 num_cols <- length(sample_ids)
 
 # Initialize an empty dense matrix
 dense_matrix <- matrix(0, nrow = num_rows, ncol = num_cols)
 
 # Populate the dense matrix column by column
 for (i in seq_along(indptr[-1])) {
   start <- indptr[i] + 1
   end <- indptr[i + 1]
   dense_matrix[indices[start:end] + 1, i] <- data[start:end]
 }
 
 # Assign row and column names
 rownames(dense_matrix) <- observation_ids
 colnames(dense_matrix) <- sample_ids
 
 # View the result
 head(dense_matrix)
 
 #making the dateset
   
 dense_df <- as.data.frame(dense_matrix)
 
#transposing the file
 transposed_df <- as.data.frame(t(dense_df))
   
  View(transposed_df)
  # Save transposed_df to a CSV file
  write.csv(transposed_df, "transposed_df_fullname.csv", row.names = TRUE)
  
  # View updated column names
  head(colnames(transposed_df))
  
