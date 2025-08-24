# YouTube Sentiment Insights

**Youtube Sentiment Insights** is a full-stack web application designed to provide a comprehensive sentiment analysis of YouTube video comments. This tool empowers content creators, marketers, and viewers to quickly gauge audience reaction and understand the overall sentiment surrounding a video's content.

By leveraging a machine learning backend and an intuitive Chrome extension frontend, this project can analyze hundreds of comments in seconds, transforming raw feedback into actionable insights through a series of clear and informative visualizations.

## Key Features

*   **On-Demand Sentiment Analysis**: Analyze comments from any YouTube video with a single click.
*   **Comprehensive Visualizations**:
    *   **Sentiment Pie Chart**: Get a quick overview of the sentiment distribution (Positive, Neutral, Negative).
    *   **Sentiment Trend Graph**: Track how audience sentiment evolves over time from the video's publication date.
    *   **Word Cloud**: Instantly identify the most frequently discussed topics and keywords in the comments.
*   **Detailed Comment Metrics**: View a summary of key statistics, including total comments analyzed, number of unique commenters, and average sentiment score.
*   **Top Comments**: See a list of the top 25 most relevant comments along with their predicted sentiment.
*   **Intuitive Chrome Extension**: A simple and easy-to-use browser extension serves as the user interface.

## System Architecture

The application consists of three main components that work together:

1.  **Chrome Extension (Frontend)**: This is the user-facing part of the application. When a user is on a YouTube video page and clicks the extension icon, it scrapes the video ID, fetches comments using the YouTube Data API, and sends them to the backend for analysis. It then receives the results and displays them in a structured and visually appealing format.

2.  **Flask API (Backend)**: A robust Python backend powered by Flask. It exposes several API endpoints to handle requests from the frontend. Its core responsibilities include:
    *   Preprocessing the raw comment text.
    *   Predicting sentiment using a pre-trained LightGBM model.
    *   Generating all visualizations (pie chart, trend graph, word cloud) on the fly.

3.  **YouTube Data API**: The official Google API used to fetch video comments reliably.

The end-to-end workflow is as follows:
`User activates Chrome Extension -> Extension calls YouTube API for comments -> Extension sends comments to Flask Backend -> Backend performs analysis & generates visualizations -> Extension displays results to the user.`

## Technology Stack

### Backend & Machine Learning
*   **Python 3.11**
*   **Flask**: For the web API.
*   **Scikit-learn**: For TF-IDF vectorization and ML pipelines.
*   **LightGBM**: For the sentiment classification model.
*   **Pandas**: For data manipulation.
*   **NLTK**: For natural language processing tasks (stopwords, lemmatization).
*   **DVC (Data Version Control)**: To create and manage the machine learning pipeline.
*   **MLflow**: Used for experiment tracking and model registry (as seen in the development history).
*   **Matplotlib & WordCloud**: For generating visualizations.
*   **Gunicorn**: As a production-ready WSGI server.

### Frontend
*   **JavaScript (ES6)**
*   **HTML5 & CSS3**
*   **Chrome Extension API (Manifest V3)**

### Infrastructure
*   **Docker**: For containerizing the Flask application.
*   **AWS (EC2/ELB)**: The original deployment target for the backend service.

## Machine Learning Pipeline

This project uses a DVC-powered pipeline to ensure reproducibility and modularity in the machine learning workflow. The pipeline automates the key stages of model development:

1.  **Data Ingestion**: Loads the raw dataset of YouTube comments.
2.  **Data Preprocessing**: Cleans and preprocesses the text data. This involves lowercasing, removing special characters, stripping whitespace, removing stopwords, and lemmatization. The processed data is then split into training and testing sets.
3.  **Model Training**: Trains the LightGBM classifier on the preprocessed training data.
4.  **Model Evaluation**: Evaluates the trained model on the test set, generating metrics like accuracy, precision, recall, F1-score, and a confusion matrix.

The entire pipeline can be executed with a single command (`dvc repro`), which ensures that every step is run in the correct order. The specifics of the pipeline are defined in `dvc.yaml`, and the parameters are managed in `params.yaml`.

The experimentation process, including baseline models and hyperparameter tuning, is documented in the Jupyter Notebooks inside the `/notebooks` directory.

## Setup and Usage

To get this project up and running locally, follow these steps.

### Prerequisites
*   [Git](https://git-scm.com/downloads)
*   [Conda](https://docs.conda.io/en/latest/miniconda.html) for environment management.
*   A valid **YouTube Data API Key**. You can obtain one from the [Google Cloud Console](https://console.cloud.google.com/).

### Backend Setup

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd Youtube_Sentiment_Insights
    ```

2.  **Create and activate the Conda environment:**
    ```bash
    conda create -n youtube python=3.11 -y
    conda activate youtube
    ```

3.  **Install the required dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Run the Flask application:**
    ```bash
    python app.py
    ```
    The backend server will now be running on `http://localhost:8080`.

### Frontend Setup

1.  **Add your API Key:**
    *   Open the file `yt-chrome-plugin-frontend/popup.js`.
    *   Replace the placeholder `AIzaSy...` with your actual YouTube Data API key in the `API_KEY` constant.
    *   Ensure the `API_URL` constant is pointing to your local backend: `http://localhost:8080/`.

2.  **Load the Chrome Extension:**
    *   Open Google Chrome and navigate to `chrome://extensions`.
    *   Enable "Developer mode" using the toggle in the top-right corner.
    *   Click on the "Load unpacked" button.
    *   Select the `yt-chrome-plugin-frontend` directory from the cloned repository.
    *   The "Youtube Sentiment Insights" extension should now appear in your list of extensions and in your browser's toolbar.

### Usage

1.  Navigate to any YouTube video page.
2.  Click the "Youtube Sentiment Insights" icon in your Chrome toolbar.
3.  The extension will activate and begin fetching and analyzing comments. The results will be displayed within a few seconds.

## API Endpoints

The Flask backend provides the following API endpoints:

#### `POST /predict_with_timestamps`
*   **Description**: Predicts the sentiment for a list of comments, including their timestamps.
*   **Request Body**:
    ```json
    {
      "comments": [
        { "text": "This is a great video!", "timestamp": "2023-10-27T10:00:00Z" },
        { "text": "I did not like it.", "timestamp": "2023-10-27T11:00:00Z" }
      ]
    }
    ```
*   **Response**:
    ```json
    [
      { "comment": "This is a great video!", "sentiment": "1", "timestamp": "2023-10-27T10:00:00Z" },
      { "comment": "I did not like it.", "sentiment": "-1", "timestamp": "2023-10-27T11:00:00Z" }
    ]
    ```

#### `POST /generate_chart`
*   **Description**: Generates a pie chart image from sentiment counts.
*   **Request Body**:
    ```json
    {
      "sentiment_counts": {
        "1": 150,
        "0": 50,
        "-1": 25
      }
    }
    ```
*   **Response**: An image file (`image/png`).

#### `POST /generate_wordcloud`
*   **Description**: Generates a word cloud image from a list of comments.
*   **Request Body**:
    ```json
    {
      "comments": ["This is a great video!", "I learned so much."]
    }
    ```
*   **Response**: An image file (`image/png`).

#### `POST /generate_trend_graph`
*   **Description**: Generates a sentiment trend graph from a list of sentiment data points.
*   **Request Body**:
    ```json
    {
      "sentiment_data": [
        { "timestamp": "2023-10-27T10:00:00Z", "sentiment": "1" },
        { "timestamp": "2023-10-28T11:00:00Z", "sentiment": "-1" }
      ]
    }
    ```
*   **Response**: An image file (`image/png`).
