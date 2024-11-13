# Mirror, Mirror: Exploring Stereotype Presence Among Top-N Recommendations Than May Reach Children

This repository contains the code to reproduce the empirical exploration accomplished in the paper:
**Mirror, Mirror: Exploring Stereotype Presence Among Top-N Recommendations Than May Reach Children**, submitted to the TORS special issue on Recommender Systems for Good. 

Here, we provide a detailed overview regarding necessary steps for reproducing the results found and discussed in this work.





## Requirements

In order to reproduce the steps of the empirical exploration, the following tools and datasets are required.

- [Elliot](https://github.com/sisinflab/elliot)
- [Movielens-1M](https://grouplens.org/datasets/movielens/1m/) dataset
- [Goodreads](https://mengtingwan.github.io/data/goodreads.html#datasets) with "children" Genre
 (`goodreads_interactions_children.json.gz` and `goodreads_books_children.json.gz`)
 - [TMDB 10000 Popular Movies](https://www.kaggle.com/datasets/muqarrishzaib/tmdb-10000-movies-dataset)
 - Add all datasets to the `data` directory

The directory should have the following structure:
```
├── Additional_Statistics
├── config_files
├── data
│   ├── ml-1m
│   │   ├── movies.dat
│   │   ├── ratings.dat
│   │   ├── users.dat
|   |   ├── README
│   ├── goodreads_interactions_children.json.gz
│   ├── goodreads_books_children.json.gz
|   ├── TMDB 10000 Movies Dataset.csv
|   ├── data_processing
├── elliot
|   ├── config_files
|   ├── data
|   ├── ...
├── Gender Stereotype Detection Methods
├── input_recommendations
├── Result_Processing
├── Results
├── ...
```



## 1. Dataset Preprocessing
To preprocess the datasets and gather the item description, please run the following scripts:
- `preprocess_data/read_goodreads.py`
- `preprocess_data/add_ML_descriptions.py`


For both datasets, the remaining preprocessing steps (including splitting and filtering) are handled by the `config_file`s and Elliot.



## 2. Run Elliot
- Install Elliot in the main directory
- Create conda environment
```
conda create --yes --name elliot-env python=3.8
conda activate elliot-env
cd Experiment_2 
git clone https://github.com//sisinflab/elliot.git && cd elliot
pip install --upgrade pip
pip install -e . --verbose
```

- Add the rating data to elliot: `elliot/data/goodreads_ratings.tsv`. Movielens data should automatically be added by installing Elliot
- Add the config files from the directory `config_files` to elliot: `elliot/config_files/reproducibility_goodreads.yml` and `elliot/config_files/reproducibility_movielens.yml`
- Generate the recommendations with Elliot (`.tsv` files, see [docs](https://elliot.readthedocs.io/en/latest/guide/introduction.html)). 


## 3. Create Evaluation Subset
- Add recommendation directories to the directory `input_recommendations` as `input_recommendations/goodreads/*.tsv` and  `input_recommendations/movielens/*.tsv`
- For ML, filter the results of Elliot by users with age < 18 (ML-children) by running `Result_Processing/filter_ML.py`


## 4. Predict stereotypes using the SDMs
- Generate files that indicate whether a stereotype was found for each item in the recommendation list.
- Create stereotype predictions for all items using each SDM for each sterotype (NGIM only Gender)

### Preparation
- By using the `data_processing/create_subset.py` script save the item descriptions as csv files with the column names: "id", "description" as
1. `data/ml_descriptions_processed.csv`
2. `data/gr_descriptions_processed.csv`

### NGIM
Compute and save the predicted scores for each item by running the script: `Gender Stereotype Detection Methods/NGIM/ngim.py`

### BiasMeter
- Get [BiasMeter](https://github.com/YacineGACI/BiasMeter) and fit the models. Add all scripts and data to `Gender Stereotype Detection methods/BiasMeter` 
- Tokenize MovieLens and Goodreads descriptions with `Gender Stereotype Detection Methods/BiasMeter/tokenize.py`
- Run `Gender Stereotype Detection Methods/BiasMeter/biasmeter_dataset_classification.py` in order to analyze the individual sentences in the descriptions.
- Run `Gender Stereotype Detection Methods/BiasMeter/calc_cf.py` in order to compute the Stanford Certainty Factor and save the predicted labels.


### ChatGPT
- Get access to chat-gpt using OpenAI's [API parallel processor](https://github.com/openai/openai-cookbook/blob/main/examples/api_request_parallel_processor.py); we used the model gpt-3.5-turbo
- Generate input prompts using `Gender Stereotype Detection Methods/GPT/chatgpt_label_datasets.py`
- Run the input prompts using the abovementioned parallel processor; Example calls:
    ```python
    python "api_request_parallel_processor.py" \
        --requests_filepath "Gender Stereotype Detection Methods/data/jobs_gr_1.jsonl" \   # Path to input JSONL requests file
        --save_filepath "Gender Stereotype Detection Methods/results/sdm_results_gr.jsonl" \ # Path to save the results
        --api_key **your_api_key** \   # API key for authentication
        --request_url "https://api.openai.com/v1/chat/completions" \  # OpenAI API endpoint
        --max_requests_per_minute 400 \    # Limit on requests per minute
        --max_tokens_per_minute 100000     # Token limit per minute
    ```
    ```python
    python "api_request_parallel_processor.py" \
        --requests_filepath "Gender Stereotype Detection Methods/data/jobs_gr_2.jsonl" \   # Path to input JSONL requests file
        --save_filepath "Gender Stereotype Detection Methods/results/sdm_results_gr.jsonl" \ # Path to save the results
        --api_key **your_api_key** \   # API key for authentication
        --request_url "https://api.openai.com/v1/chat/completions" \  # OpenAI API endpoint
        --max_requests_per_minute 400 \    # Limit on requests per minute
        --max_tokens_per_minute 100000     # Token limit per minute
    ```
    ```python
    python "api_request_parallel_processor.py" \
        --requests_filepath "Gender Stereotype Detection Methods/data/jobs_ml.jsonl" \   # Path to input JSONL requests file
        --save_filepath "Gender Stereotype Detection Methods/results/sdm_results_ml.jsonl" \ # Path to save the results
        --api_key **your_api_key** \   # API key for authentication
        --request_url "https://api.openai.com/v1/chat/completions" \  # OpenAI API endpoint
        --max_requests_per_minute 400 \    # Limit on requests per minute
        --max_tokens_per_minute 100000     # Token limit per minute
    ```
- Predict a stereotype if GPT's stereotype probability is > 0.7 by running the script `Gender Stereotype Detection Methods/GPT/chatgpt_read_results.py`

### Processing Results
- Run `Gender Stereotype Detection Methods/Genesis.py` in order to summarize the results from the different metrics.


## 5. Stereotype Evaluation
- Run `calculate_stereotype_metrics.py` and make sure the path to the recommendations are correct (**lines 244 & 245**).
- All the metrics for each RA will be stored as *.csv* files in `Results/SDM/...`
- Run `plot_metrics.py` after the results are stored to plot the metrics.

- Remark that we re-run the statistics with `R`. Those scripts can be found in `Additional_Statistical_Eval`





