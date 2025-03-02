---
title: 'Tutorial: Predict prices using regression with Model Builder'
description: This tutorial illustrates how to build a regression model using ML.NET Model Builder to predict prices, specifically, New York City taxi fares.
author: luisquintanilla
ms.author: luquinta
ms.date: 09/18/2019
ms.topic: tutorial
ms.custom: mvc
#Customer intent: As a non-developer, I want to use Model Builder to automatically generate a model to predict prices using Model Builder.
---

# Tutorial: Predict prices using regression with Model Builder

Learn how to use ML.NET Model Builder to build a regression model() to predict prices.  The .NET console app that you develop in this tutorial predicts taxi fares based on historical New York taxi fare data.

The Model Builder price prediction template can be used for any scenario requiring a numerical prediction value. Example scenarios include: house price prediction, demand prediction, and sales forecasting.

In this tutorial, you learn how to:
> [!div class="checklist"]
>
> - Prepare and understand the data
> - Choose a scenario
> - Load the data
> - Train the model
> - Evaluate the model
> - Use the model for predictions

> [!NOTE]
> Model Builder is currently in Preview.

## Pre-requisites

For a list of pre-requisites and installation instructions, visit the [Model Builder installation guide](../how-to-guides/install-model-builder.md).

## Create a console application

1. Create a **.NET Core Console Application** called "TaxiFarePrediction".

## Prepare and understand the data

1. Create a directory named *Data* in your project to store the data set files.

1. The data set used to train and evaluate the machine learning model is originally from the NYC TLC Taxi Trip data set.

    To download the data set, navigate to the [taxi-fare-train.csv download link](https://raw.githubusercontent.com/dotnet/machinelearning/master/test/data/taxi-fare-train.csv).

    When the page loads, right-click anywhere on the page and select **Save as**.

    Use the **Save As Dialog** to save the file in the *Data* folder you created at the previous step.

1. In **Solution Explorer**, right-click the *taxi-fare-train.csv* file and select **Properties**. Under **Advanced**, change the value of **Copy to Output Directory** to **Copy if newer**.

Each row in the `taxi-fare-train.csv` data set contains details of trips made by a taxi.

1. Open the **taxi-fare-train.csv** data set

    The provided data set contains the following columns:

    - **vendor_id:** The ID of the taxi vendor is a feature.
    - **rate_code:** The rate type of the taxi trip is a feature.
    - **passenger_count:** The number of passengers on the trip is a feature.
    - **trip_time_in_secs:** The amount of time the trip took.
    - **trip_distance:** The distance of the trip is a feature.
    - **payment_type:** The payment method (cash or credit card) is a feature.
    - **fare_amount:** The total taxi fare paid is the label.

The `label` is the column you want to predict. When performing a regression task, the goal is to predict a numerical value. In this price prediction scenario, the cost of a taxi ride is being predicted. Therefore, the **fare_amount** is the label. The identified `features` are the inputs you give the model to predict the `label`. In this case, the rest of the columns are used as features or inputs to predict the fare amount.

## Choose a scenario

To train your model, you need to select from the list of available machine learning scenarios provided by Model Builder. In this case, the scenario is `Price Prediction`.

1. In **Solution Explorer**, right-click the *TaxiFarePrediction* project, and select **Add** > **Machine Learning**.
1. In the scenario step of the Model Builder tool, select *Price Prediction* scenario.

## Load the data

Model Builder accepts data from two sources, a SQL Server database or a local file in csv or tsv format.

1. In the data step of the Model Builder tool, select *File* from the data source dropdown.
1. Select the button next to the *Select a file* text box and use File Explorer to browse and select the *taxi-fare-test.csv* in the *Data* directory
1. Choose *fare_amount* in the *Label or Column to Predict* dropdown and navigate to the train step of the Model Builder tool.

## Train the model

The machine learning task used to train the price prediction model in this tutorial is regression. During the model training process, Model Builder trains separate models using different regression algorithms and settings to find the best performing model for your dataset.

The time required for the model to train is proportionate to the amount of data. Model Builder automatically selects a default value for **Time to train (seconds)** based on the size of your data source.

1. Leave the default value as is for *Time to train (seconds)* unless you prefer to train for a longer time.
2. Select *Start Training*.

Throughout the training process, progress data is displayed in the `Progress` section of the train step.

- Status displays the completion status of the training process.
- Best accuracy displays the accuracy of the best performing model found by Model Builder so far. Higher accuracy means the model predicted more correctly on test data.
- Best algorithm displays the name of the best performing algorithm performed found by Model Builder so far.
- Last algorithm displays the name of the algorithm most recently used by Model Builder to train the model.

Once training is complete, navigate to the evaluate step.

## Evaluate the model

The result of the training step will be one model which had the best performance. In the evaluate step of the Model Builder tool, the output section, will contain the algorithm used by the best performing model in the *Best Model* entry along with metrics in *Best Model Quality (RSquared)*. Additionally, a summary table containing top five models and their metrics.

If you're not satisfied with your accuracy metrics, some easy ways to try and improve model accuracy are to increase the amount of time to train the model or use more data. Otherwise, navigate to the code step.

## Add the code to make predictions

Two projects will be created as a result of the training process.

- TaxiFarePredictionML.ConsoleApp: A .NET Core Console application that contains the model training and consumption code.
- TaxiFarePredictionML.Model: A .NET Standard class library containing the data models that define the schema of input and output model data as well as the persisted version of the best performing model during training.

1. In the code step of the Model Builder tool, select **Add Projects** to add the auto-generated projects to the solution.
1. Right-click *TaxiFarePrediction* project. Then, **Add > Reference**. Choose the **Projects > Solution** node and from the list, check the *TaxiFarePredictionML.Model* project and select OK.
1. Open the *Program.cs* file in the *TaxiFarePrediction* project.
1. Add the following using statements to reference the *Microsoft.ML* NuGet package and *TaxiFarePredictionML.Model* project:

    ```csharp
    using System;
    using Microsoft.ML;
    using TaxiFarePredictionML.Model.DataModels;
    ```

1. Add the `ConsumeModel` method to the `Program` class.

    ```csharp
    static ModelOutput ConsumeModel(ModelInput input)
    {
        // 1. Load the model
        MLContext mlContext = new MLContext();
        ITransformer mlModel = mlContext.Model.Load("MLModel.zip", out var modelInputSchema);

        // 2. Create PredictionEngine
        var predictionEngine = mlContext.Model.CreatePredictionEngine<ModelInput, ModelOutput>(mlModel);

        // 3. Use PredictionEngine to use model on input data
        ModelOutput result = predictionEngine.Predict(input);

        // 4. Return prediction result
        return result;
    }
    ```

    The `ConsumeModel` will load the trained model, create a [`PredictionEngine`](xref:Microsoft.ML.PredictionEngine%602) for the model and use it to make predictions on new data.

1. To make a prediction on new data using the model, create a new instance of the `ModelInput` class and use the `ConsumeModel` method. Notice that the fare amount is not part of the input. This is because the model will generate the prediction for it. Add the following code to the `Main` method and run the application

    ```csharp
    // Create sample data
    ModelInput input = new ModelInput()
    {
        Vendor_id = "CMT",
        Rate_code = 1,
        Passenger_count = 1,
        Trip_time_in_secs = 1271,
        Trip_distance = 3.8f,
        Payment_type = "CRD"
    };

    // Make prediction
    ModelOutput prediction = ConsumeModel(input);

    // Print Prediction
    Console.WriteLine($"Predicted Fare: {prediction.Score}");
    Console.ReadKey();
    ```

    The output generated by the program should look similar to the snippet below:

    ```bash
    Predicted Fare: 16.82245
    ```

If you need to reference the generated projects at a later time inside of another solution, you can find them inside the `C:\Users\%USERNAME%\AppData\Local\Temp\MLVSTools` directory.

## Next Steps

In this tutorial, you learned how to:
> [!div class="checklist"]
>
> - Prepare and understand the data
> - Choose a scenario
> - Load the data
> - Train the model
> - Evaluate the model
> - Use the model for predictions

### Additional Resources

To learn more about topics mentioned in this tutorial, visit the following resources:

- [Model Builder Scenarios](../automate-training-with-model-builder.md#scenarios)
- [Regression](../resources/glossary.md#regression)
- [Regression Model Metrics](../resources/metrics.md#metrics-for-regression)
- [NYC TLC Taxi Trip data set](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml)
