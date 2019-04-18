---
title: "Deploy a vision AI model with Visual Studio Code"
permalink: /docs/Deploy_model_VS_Code/
excerpt: "Using Visual Studio Code deployment to deploy models to the Vision AI DevKit hardware."
variable:
  - platform: windows
    name: Windows
  - platform: macos
    name: macOS
last_modified_at: 2019-04-11
---

## What you will do
* Configure VS Code for model deployment
* Deploy a pre-built sample model container image to the Visual AI DevKit hardware

## What you will need
* Visual Studio (VS) Code and subsidary tools [(Instructions)](https://azure.github.io/Vision-AI-DevKit-Pages/docs/SetUp_VS_Code/){:target="_blank"}
* Vision Dev Kit sample model - [GitHub repository](https://github.com/Microsoft/vision-ai-developer-kit/tree/master/sample-solutions/VisionSample).
* Azure IoT Hub and Azure IoT Edge device configured for your Vision AI DevKit hardware [(Instructions)]({{ '/docs/Setup_Azure_resources/' | relative_url}})
* Active Wi-Fi access point with Internet connectivity.
* (optional) HDMI cable and monitor to view video from the Visual AI DevKit.

## Deploy the sample model container image to the Vision AI DevKit

1. Download or Clone the latest Vision AI DevKit repository from [GitHub](https://github.com/Microsoft/vision-ai-developer-kit/tree/master/sample-solutions/VisionSample).
2. Launch Visual Studio Code, and select [File > Open Folder…] command to open the “VisionSample” directory as the workspace root.
3. Use the [Python: Select Interpreter] command in the command palette box or click the current “Python interpreter” option on the bottom line to set "python.pythonPath" in .vscode\settings.json.
![Setting the pythonPath screen shot]({{ '/assets/images/VSC_Deploy_Python37.JPG' | relative_url }}) 
4. Open “00-aml-configuration.py” under “VisionSample\MachineLearning\scripts” folder and replace the following 3 default settings with your Azure Machine Learning Service Workspace settings:
![Setting the pythonPath screen shot]({{ '/assets/images/VSC_Deploy_mod_00-aml-configuration.py_screenshot.png' | relative_url }})
5. Click the [Run Cell] or [Run All Cells] link on the top line of the cell. A new workspace will be created, if one does not already exist, along with a “config.json” file under the “VisionSample\aml_config” folder.  
![Modify config.json screen shot]({{ '/assets/images/VSC_Deploy_config-json_screenshot.png' | relative_url }}) 
6. Open “01-convert-model-containerize.py” under the “VisionSample\MachineLearning\scripts” folder and click the [Run Cell] or [Run All Cells] link to: register the model, convert the model, create a container image, and write settings related to the container image to the “.env” file under the “VisionSample\EdgeSolution” folder.
  * Note: This .py script will import *current_config.py* under the *VisionSample\MachineLearning\scripts\model_configs* folder to process the model specified in *current_config.py*.  The script can be reused to process a different model by changing *current_config.py’s* content. The initial content in *current_config.py* is for the pre-trained model *imagenet_2_frozen.pb* located in the *VisionSample\MachineLearning\models\mobilenet-imagenet\orig* folder.
![Initial contents of current_config.json screen shot]({{ '/assets/images/VSC_Deploy_current-config-py_Initial_content_screenshot.png' | relative_url }})
7. Right click the *deployment.template.json* file under the *VisionSample\DeployContainerFromAML* folder and select **Generate IoT Edge Deployment Manifest** to create a new *deployment.json* file under *VisionSample\DeployContainerFromAML\config* folder.  
![Open deployment.template.json screen shot]({{ '/assets/images/VSC_Deploy_open_deployment-template-json_screenshot.png' | relative_url }}) 
8. Click the [Explorer] icon, then click […] in the **AZURE IOT HUB DEVICES** section on the right side. Click **Select IoT Hub** to select your Azure IoT Hub.
9. Expand the **AZURE IOT HUB DEVICES** section. Right-click your IoT Edge device, then select **Create Deployment for Single Device**. Click the *deployment.json* file under the *VisionSample\DeployContainerFromAML\config* folder, then click the **Select Edge Deployment Manifest** button to deploy the container image to the IoT Edge device (your Vision AI DevKit hardware).
![Open deployment.template.json from VSC Screenshot]({{ '/assets/images/VSC_Deploy_Select-Edge-Deployment-Manifest_screenshot.png' | relative_url }})  
 ![Open deployment.template.json from VSC Screenshot #2]({{ '/assets/images/VSC_Deploy_Select-Edge-Deployment-Manifest_screenshot2.png' | relative_url }})
10.	Configure the device as an IoT Edge device using the connection string for your Vision AI DevKit, obtained earlier.
11.	Monitor the deployment status for the AI Vision Kit device by the command lines
```
adb shell docker ps
adb shell docker logs edgeAgent
```
![Use ADB to monitor deployment status Screenshot]({{ '/assets/images/VSC_Deploy_Using_ADB_to_monitor_deployment_screenshot.png' | relative_url }})
12.	Check object detection results using the command line
```
adb shell iotedge logs <module name>
```
(e.g. “iotedge logs VisionSampleImagenet”.)
Mobilenet-imagenet model can detect the 1000 object classes listed in the  *VisionSample\MachineLearning\models\mobilenet-imagenet\orig\output_labels.txt*.

    ![Use ADB to module impage outputs Screenshot]({{ '/assets/images/VSC_Deploy_Using_ADB_to_check_detection_results_screenshot.png' | relative_url }})

In VS Code, select **AZURE IOT HUB DEVICES > … > Select IoT Hub** and then **AZURE IOT HUB DEVICES > … > Start Monitoring D2C Message** command to monitor the messages sent from the AI Vision DevKit to your Azure IoT Hub.
    ![Screenshot of message monitoring output]({{ '/assets/images/VSC-Deploy-Monitor-messages.png' | relative_url }})

### Verify output on the monitor
Connect the HDMI cable to the Vision AI DevKit hardware and your monitor. A few minutes after submitting your deployment, you should start to see video on the monitor with bounding boxes drawing around objects the camera can see and evaluate.

## Retraining the machine learning model
1. Open *02-mobilenet-transfer-learning-cloud.py* and click **Run Cell** or **Run All Cells** to train a new mobilenet model in the cloud using *soda_cans* data under the *VisionSample\MachineLearning\data\soda_cans* folder.
2. After **Run All Cells** execution has completed, it will write a *va-snpe-engine-library_config.json* config file to the new model folder, the write the model config file to *VisionSample\MachineLearning\scripts\model_configs* and overwrite *current_config.py*.
3. Repeat steps 6 and 7 in the previous section to open and execute *01-convert-model-containerize.py*, using the newly generated *deployment.json* to deploy the new model image *retrained_graph.pb* under the *VisionSample\MachineLearning\models\mobilenet-retrain-cloud\outputs* folder.