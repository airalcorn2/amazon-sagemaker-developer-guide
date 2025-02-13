# SageMaker Debugger Interactive Report for XGBoost<a name="debugger-report-xgboost"></a>

Receive training reports autogenerated by Debugger\. The Debugger reports provide insights into your training jobs and suggest recommendations to improve your model performance\.

**Note**  
You can download a Debugger reports while your training job is running or after the job has finished\. During training, Debugger concurrently updates the report reflecting the current rules' evaluation status\. You can download a complete Debugger report only after the training job has completed\.

**Important**  
In the report, plots and and recommendations are provided for informational purposes and are not definitive\. You are responsible for making your own independent assessment of the information\.

## SageMaker Debugger XGBoost Training Report<a name="debugger-training-xgboost-report"></a>

For SageMaker XGBoost training jobs, use the Debugger [CreateXgboostReport](debugger-built-in-rules.md#create-xgboost-report) rule to receive a comprehensive training report of the training progress and results\. Following this guide, specify the [CreateXgboostReport](debugger-built-in-rules.md#create-xgboost-report) rule while constructing an XGBoost estimator, download the report using the [Amazon SageMaker Python SDK](https://sagemaker.readthedocs.io) or the Amazon S3 console, and gain insights into the training results\.

**Important**  
In the report, plots and and recommendations are provided for informational purposes and are not definitive\. You are responsible for making your own independent assessment of the information\.

**Topics**
+ [Construct a SageMaker XGBoost Estimator with the Debugger XGBoost Report Rule](#debugger-training-xgboost-report-estimator)
+ [Download the Debugger XGBoost Training Report](#debugger-training-xgboost-report-download)
+ [Debugger XGBoost Training Report Walkthrough](#debugger-training-xgboost-report-walkthrough)

### Construct a SageMaker XGBoost Estimator with the Debugger XGBoost Report Rule<a name="debugger-training-xgboost-report-estimator"></a>

The [CreateXgboostReport](debugger-built-in-rules.md#create-xgboost-report) rule collects the following output tensors from your training job: 
+ `hyperparameters` – Saves at the first step\.
+ `metrics` – Saves loss and accuracy every 5 steps\.
+ `feature_importance` – Saves every 5 steps\.
+ `predictions` – Saves every 5 steps\.
+ `labels` – Saves every 5 steps\.

The output tensors are saved at a default S3 bucket\. For example, `s3://sagemaker-<region>-<12digit_account_id>/<base-job-name>/debug-output/`\.

When you construct a SageMaker estimator for an XGBoost training job, specify the rule as shown in the following example code\.

------
#### [ Using the SageMaker generic estimator ]

```
import boto3
import sagemaker
from sagemaker.estimator import Estimator
from sagemaker import image_uris
from sagemaker.debugger import Rule, rule_configs

rules=[
    Rule.sagemaker(rule_configs.create_xgboost_report())
]

region = boto3.Session().region_name
xgboost_container=sagemaker.image_uris.retrieve("xgboost", region, "1.2-1")

estimator=Estimator(
    role=sagemaker.get_execution_role()
    image_uri=xgboost_container,
    base_job_name="debugger-xgboost-report-demo",
    instance_count=1,
    instance_type="ml.m5.2xlarge",
    
    # Add the Debugger XGBoost report rule
    rules=rules
)

estimator.fit(wait=False)
```

------

### Download the Debugger XGBoost Training Report<a name="debugger-training-xgboost-report-download"></a>

Download the Debugger XGBoost training report while your training job is running or after the job has finished using the [Amazon SageMaker Python SDK](https://sagemaker.readthedocs.io) and AWS Command Line Interface \(CLI\)\.

------
#### [ Download using the SageMaker Python SDK and AWS CLI ]

1. Check the current job's default S3 output base URI\.

   ```
   estimator.output_path
   ```

1. Check the current job name\.

   ```
   estimator.latest_training_job.job_name
   ```

1. The Debugger XGBoost report is stored under `<default-s3-output-base-uri>/<training-job-name>/rule-output`\. Configure the rule output path as follows:

   ```
   rule_output_path = estimator.output_path + "/" + estimator.latest_training_job.job_name + "/rule-output"
   ```

1. To check if the report is generated, list directories and files recursively under the `rule_output_path` using `aws s3 ls` with the `--recursive` option\.

   ```
   ! aws s3 ls {rule_output_path} --recursive
   ```

   This should return a complete list of files under autogenerated folders that are named `CreateXgboostReport` and `ProfilerReport-1234567890`\. The XGBoost training report is stored in the `CreateXgboostReport`, and the profiling report is stored in the `ProfilerReport-1234567890` folder\. To learn more about the profiling report generated by default with the XGBoost training job, see [SageMaker Debugger Profiling Report](debugger-report.md#debugger-profiling-report)\.  
![\[An example of rule output.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-xgboost-report-ls.png)

   The `xgboost_report.html` is an autogenerated XGBoost training report by Debugger\. The `xgboost_report.ipynb` is a Jupyter notebook that's used to aggregate training results into the report\. You can download all of the files, browse the HTML report file, and modify the report using the notebook\.

1. Download the files recursively using `aws s3 cp`\. The following command saves all of the rule output files to the `ProfilerReport-1234567890` folder under the current working directory\.

   ```
   ! aws s3 cp {rule_output_path} ./ --recursive
   ```
**Tip**  
If you are using a Jupyter notebook server, run `!pwd` to verify the current working directory\.

1. Under the `/CreateXgboostReport` directory, open `xgboost_report.html`\. If you are using JupyterLab, choose **Trust HTML** to see the autogenerated Debugger training report\.  
![\[An example of rule output.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-xgboost-report-open-trust.png)

1. Open the `xgboost_report.ipynb` file to explore how the report is generated\. You can customize and extend the training report using the Jupyter notebook file\.

------
#### [ Download using the Amazon S3 console ]

1. Sign in to the AWS Management Console and open the Amazon S3 console at [https://console\.aws\.amazon\.com/s3/](https://console.aws.amazon.com/s3/)\.

1. Search for the base S3 bucket\. For example, if you haven't specified any base job name, the base S3 bucket name should be in the following format: `sagemaker-<region>-111122223333`\. Look up the base S3 bucket through the **Find bucket by name** field\.  
![\[The Find bucket by name field in the Amazon S3 console.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-report-download-s3console-0.png)

1. In the base S3 bucket, look up the training job name by entering your job name prefix in **Find objects by prefix** and then choosing the training job name\.  
![\[The Find objects by prefix field in the Amazon S3 console.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-report-download-s3console-1.png)

1. In the training job's S3 bucket, choose **rule\-output/** subfolder\. There must be three subfolders for training data collected by Debugger: **debug\-output/**, **profiler\-output/**, and **rule\-output/**\.   
![\[An example to the rule output S3 bucket URI.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-report-download-s3console-2.png)

1. In the **rule\-output/** folder, choose the **CreateXgboostReport/** folder\. The folder contains **xbgoost\_report\.html** \(the autogenerated report in html\) and **xbgoost\_report\.ipynb** \(a Jupyter notebook with scripts that are used for generating the report\)\.

1. Choose the **xbgoost\_report\.html** file, choose **Download actions**, and then choose **Download**\.  
![\[An example to the rule output S3 bucket URI.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-xgboost-report-s3-download.png)

1. Open the downloaded **xbgoost\_report\.html** file in a web browser\.

------

### Debugger XGBoost Training Report Walkthrough<a name="debugger-training-xgboost-report-walkthrough"></a>

This section walks you through the Debugger XGBoost training report\. The report is automatically aggregated depending on the output tensor regex, recognizing what type of your training job is among binary classification, multiclass classification, and regression\.

**Important**  
In the report, plots and and recommendations are provided for informational purposes and are not definitive\. You are responsible for making your own independent assessment of the information\.

**Topics**
+ [Distribution of True Labels of the Dataset](#debugger-training-xgboost-report-walkthrough-dist-label)
+ [Loss versus Step Graph](#debugger-training-xgboost-report-walkthrough-loss-vs-step)
+ [Feature Importance](#debugger-training-xgboost-report-walkthrough-feature-importance)
+ [Confusion Matrix](#debugger-training-xgboost-report-walkthrough-confusion-matrix)
+ [Evaluation of the Confusion Matrix](#debugger-training-xgboost-report-walkthrough-eval-conf-matrix)
+ [Accuracy Rate of Each Diagonal Element Over Iteration](#debugger-training-xgboost-report-walkthrough-accuracy-rate)
+ [Receiver Operating Characteristic Curve](#debugger-training-xgboost-report-walkthrough-rec-op-char)
+ [Distribution of Residuals at the Last Saved Step](#debugger-training-xgboost-report-walkthrough-dist-residual)
+ [Absolute Validation Error per Label Bin Over Iteration](#debugger-training-xgboost-report-walkthrough-val-error-per-label-bin)

#### Distribution of True Labels of the Dataset<a name="debugger-training-xgboost-report-walkthrough-dist-label"></a>

This histogram shows the distribution of labeled classes \(for classification\) or values \(for regression\) in your original dataset\. Skewness in your dataset could contribute to inaccuracies\. This visualization is available for the following model types: binary classification, multiclassification, and regression\.

![\[An example of a distribution of true labels of the dataset graph.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-dist-label.png)

#### Loss versus Step Graph<a name="debugger-training-xgboost-report-walkthrough-loss-vs-step"></a>

This is a line chart that shows the progression of loss on training data and validation data throughout training steps\. The loss is what you defined in your objective function, such as mean squared error\. You can gauge whether the model is overfit or underfit from this plot\. This section also provides insights that you can use to determine how to resolve the overfit and underfit problems\. This visualization is available for the following model types: binary classification, multiclassification, and regression\. 

![\[An example of a loss versus step graph.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-loss-vs-step.png)

#### Feature Importance<a name="debugger-training-xgboost-report-walkthrough-feature-importance"></a>

There are three different types of feature importance visualizations provided: Weight, Gain and Coverage\. We provide detailed definitions for each of the three in the report\. Feature importance visualizations help you learn what features in your training dataset contributed to the predictions\. Feature importance visualizations are available for the following model types: binary classification, multiclassification, and regression\. 

![\[An example of a feature importance graph.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-feature-importance.png)

#### Confusion Matrix<a name="debugger-training-xgboost-report-walkthrough-confusion-matrix"></a>

This visualization is only applicable to binary and multiclass classification models\. Accuracy alone might not be sufficient for evaluating the model performance\. For some use cases, such as healthcare and fraud detection, it’s also important to know the false positive rate and false negative rate\. A confusion matrix gives you the additional dimensions for evaluating your model performance\.

![\[An example of confusion matrix.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-confusion-matrix.png)

#### Evaluation of the Confusion Matrix<a name="debugger-training-xgboost-report-walkthrough-eval-conf-matrix"></a>

This section provides you with more insights on the micro, macro, and weighted metrics on precision, recall, and F1\-score for your model\.

![\[Evaluation of the confusion matrix.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-eval-conf-matrix.png)

#### Accuracy Rate of Each Diagonal Element Over Iteration<a name="debugger-training-xgboost-report-walkthrough-accuracy-rate"></a>

This visualization is only applicable to binary classification and multiclass classification models\. This is a line chart that plots the diagonal values in the confusion matrix throughout the training steps for each class\. This plot shows you how the accuracy of each class progresses throughout the training steps\. You can identify the under\-performing classes from this plot\. 

![\[An example of an accuracy rate of each diagonal element over iteration graph.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-accuracy-rate.gif)

#### Receiver Operating Characteristic Curve<a name="debugger-training-xgboost-report-walkthrough-rec-op-char"></a>

This visualization is only applicable to binary classification models\. The Receiver Operating Characteristic curve is commonly used to evaluate binary classification model performance\. The y\-axis of the curve is True Positive Rate \(TPF\) and x\-axis is false positive rate \(FPR\)\. The plot also displays the value for the area under the curve \(AUC\)\. The higher the AUC value, the more predictive your classifier\. You can also use the ROC curve to understand the trade\-off between TPR and FPR and identify the optimum classification threshold for your use case\. The classification threshold can be adjusted to tune the behavior of the model to reduce more of one or another type of error \(FP/FN\)\.

![\[An example a receiver operating characteristic curve graph.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-rec-op-char.png)

#### Distribution of Residuals at the Last Saved Step<a name="debugger-training-xgboost-report-walkthrough-dist-residual"></a>

This visualization is a column chart that shows the residual distributions in the last step Debugger captures\. In this visualization, you can check whether the residual distribution is close to normal distribution that’s centered at zero\. If the residuals are skewed, your features may not be sufficient for predicting the labels\. 

![\[An example of a distribution of residuals at the last saved step graph.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-dist-residual.png)

#### Absolute Validation Error per Label Bin Over Iteration<a name="debugger-training-xgboost-report-walkthrough-val-error-per-label-bin"></a>

This visualization is only applicable to regression models\. The actual target values are split into 10 intervals\. This visualization shows how validation errors progress for each interval throughout the training steps in line plots\. Absolute validation error is the absolute value of difference between prediction and actual during validation\. You can identify the underperforming intervals from this visualization\.

![\[An example an absolute validation error per label bin over iteration graph.\]](http://docs.aws.amazon.com/sagemaker/latest/dg/images/debugger/debugger-training-xgboost-report-walkthrough-val-error-per-label-bin.png)