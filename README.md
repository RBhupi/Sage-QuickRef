# Quick Reference to Building a Sage-Waggle Plugin
A concise guide titled that provides quick instructions to create a plugin for the Sage node on the Waggle platform. It covers various aspects of plugin development such as creating an application, testing the plugin, scheduling scripts, and downloading data.
###### tags: `sage` `waggle`

[toc]

## To Do
- [ ] Collect key links in the top most section with a short description.
- [ ] Provide notes on  `how to get ip/serialUSB port for their devices`.
- [ ] Scheduling section needs cleaning up.
- [x] Remove longer hightlighted text and use emoji.
- [ ] We can provide links for the longer scripts that requires download.



## Disclaimer 
:warning: This is not a complete guide to plugin making. It's a collection of my personal notes and slack conversations with Cyberinfrastruture Team. This is my quick reference to copy-paste commands while working on plugins and to troubleshoot in testing/scheduling stage. Please consult the official Waggle-Sage :green_book:[Plugin Tutorials](https://docs.waggle-edge.ai/docs) for detailed guidance.


## TIPS:
Plugin=App
:point_up: Do not make a plugin from scratch unless you have to. Copy the most relavant pluging and use that as a template.


## Components of a Plugin
A Sage plugin consists of several components  which are described below:
### 1. An Application
This is just your usual python program at this point. This can be a single .py script or a set of directories with many components (e.g. ML models, unit tests, test data etc) as per the application.

:point_right: First do this step on your machine and perfect it untill your are happy with the core functionality.

`app/app.py*`
: the main Python file (sometimes also named as `main.py`) contains the code that defines the functionality of the plugin or calls other scripts to do tasks. It usually has `from waggle.plugin import Plugin` call to get the data from in-built sensors and publishes the output.

> Install [pywaggle](https://github.com/waggle-sensor/pywaggle) `pip3 install -U 'pywaggle[all]'` 


`app/test.py`
: optinal but reccomended file, contains the unit tests for the plugin. 

### 2. Dockerizing the App

:point_right: Put everything in a Docker container using waggle base image and make it work. This may require some work if libraries are not compatible. Always use latest base images from link????? 

`Dockerfile*`
: contains instructions for building a Docker image for the plugin. It specifies the waggle base image from [dockerhub], sets up the environment, installs dependencies, and sets the ==entrypoint== for the container.

`requirements.txt*`
: lists the Python dependencies for the plugin. It is used by the Dockerfile to install the required packages using `pip`.

`build.sh`
: is a optional shell script to automate building the complecated Docker image with tags etc.

`Makefile`
: optional but reccomended file includes commands for building the Docker image, running tests, and deploying the plugin.

### 3. ECR Configs and Docs
You can do this step (==except sage.yaml==) after testing on node but before the ERC submission. :smile: 

`sage.yaml*`
: is the configuration file usefull for ECR and job submission. Most importatntly it specifies the version and input arguments. 

`README.md` and `ecr-meta/ecr-science-description.md*`
: a Markdown files describing the scientific rational of the plugin as an extended abstract. This includes a description of the plugin, installation instructions, usage examples, data downloading code snippets and other relevant information.
:bulb: Keep same text in both the files and follow the template of `ecr-science-description.md`. 

`ecr-meta/ecr-icon.jpg`
: is an icon for the plugin in the Sage portal.


`ecr-meta/ecr-science-image.jpg`
: is a key image or figure plot that best represents the scientific output of the plugin. 


:green_book: Check Sage Tuorial [Part1](https://docs.waggle-edge.ai/docs/tutorials/edge-apps/intro-to-edge-apps) and [Part2](https://docs.waggle-edge.ai/docs/tutorials/edge-apps/creating-an-edge-app)

## Getting access to the node
:green_book: See [Sage Tuorial: Part 3](https://docs.waggle-edge.ai/docs/tutorials/edge-apps/testing-an-edge-app) for details on this topic. 


1. Create ssh public key using 
`cd ~/.ssh` and `ssh-keygen -b 2048 -t rsa` and entering a passphrase for the private key. You will copy-paste the content of the <id_rsa.pub> in below steps.


3. Follow this page: https://auth.sagecontinuum.org/update-my-keys to access the nodes, except for the Waggle private key.
4. The ssh config required can be found on the same page, except for the Waggle private key which can be obtained by contacting the team :hourglass_flowing_sand:.
5. Sample config file is shown below


```bash
PreferredAuthentications publickey,keyboard-interactive,password

Host waggle-dev-sshd
    HostName 192.5.86.5
    Port 49190
    User waggle
    IdentityFile ~/.ssh/id_rsa # <<< your personal key
    IdentitiesOnly yes

Host waggle-dev-node-*
    ProxyCommand ssh waggle-dev-sshd connect-to-node $(echo %h | sed "s/waggle-dev-node-//" )
    User waggle
    IdentityFile ~/.ssh/ecdsa_waggle_dev # <<< waggle dev key. please contact us for this.
    IdentitiesOnly yes
    StrictHostKeyChecking no
```

5. To test your connection, execute `ssh waggle-dev-sshd` and enter _your ssh key passphrase_. You should get the following output,


> Enter passphrase for key '/Users/bhupendra/.ssh/id_rsa':
> no command provided
> Connection to 192.5.86.5 closed.
> 
> Enter the passphrase to continue.


6. To connect to the node, execute `ssh waggle-dev-node-V032` and enter the _passphrase provided by the waggle-dev_.

You should see the following message,

> Welcome to our node SSH gateway, Bhupendra Raut!
> We are connecting you to node V030 

## Testing plugins on the nodes
:green_book: For more details on this topic check [pluginctl docs](https://github.com/waggle-sensor/edge-scheduler/tree/main/docs/pluginctl).

### 1. Download and Run it

#### Download
- If you have not already done it, you need your plugin into a public github repository at this stage.
- To test the app on a node, go to nodes W0xx (e.g. W023) and clone your repo there using the command `git clone`.

- At this stage, you can play with your plugin in the docker container untill you are happy. Then if there are changes, do `git commit -am 'changes from node'` and `git push -u origin main`
:warning: Make sure your `Dockerfile` has a proper **entrypoint** all the time or the `pluginctl` run will fail.

#### Testing with Pluginctl
1. Then to test execute the command `sudo pluginctl build .`. This will output the plugin-image registry address at the end of the build. Example: `10.31.81.1:5000/local/my-plugin-name`
4. To run the plugin without input argument, use `sudo pluginctl run -n <some-unique-name> <1x.xx.xx.x:5000/local/my-plugin-name>`
5.  Execute the command with input arguments. `sudo pluginctl run -n <some-unique-name> <1x.xx.xx.x:5000/local/my-plugin-name> -- -i top_camera`. 
7.  IF you need GPU, use the selector `sudo pluginctl run -n <some-unique-name> <1x.xx.xx.x:5000/local/my-plugin-name> -- -i top_camera`.
8. :exclamation: `--` is a separator. After the `--` all arguments are for your entrypoint i.e. app.py.
9. To check running plugins, execute `sudo pluginctl ps`. To stop the plugin, execute `sudo pluginctl rm` cloud-motion.
:warning:==Do not forget to stop the plugins== after testing or it will run forever. 

### 2. Check if it worked
Login to the Sage portal and follow the instructions from section [See Your Data on Sage Portal](https://hackmd.io/iKUbbfZ7RYGbXT_tnBd1vg?both#See-Your-Data-on-Sage-Portal)

### 3. Troubleshooting Failed Plugin
When you encounter a failing/long pending job with an error, you can use the following steps to help you diagnose the issue:

1. Use the command `sudo kubectl get pod` to get the name of the pod associated with the failing job.
2. Use the command `sudo kubectl logs <<POD_NAME>>` to display the logs for the pod associated with the failing job. These logs will provide you with information on why the job failed.
3. Use the command `sudo kubectl describe pod <<POD_NAME>>` to display detailed information about the pod associated with the failing job. 
4. This information can help you identify any issues with the pod itself, such as issues with its configuration or resources.

By following these steps, you can better understand why the job is failing and take steps to resolve the issue.


## Edge code repository
### How to Get Your Plugin on ECR

To publish your Plugin on ECR, follow these steps:
1. Go to https://sagecontinuum.org/apps/.
2. Click on "Explore the Apps Portal".
3. Click on "My Apps". You must be logged in to continue.
4. Click "Create App" and enter your Github Repo URL.
5. 'Click "Register and Build App".
6. After the build process is complete, you need to ==make the app public==.
7. On Your app page click on the "Tags" tab to get the registry link when you need run the job on the node either using pluginctl or job script. This will look like:
`docker pull registry.sagecontinuum.org/bhupendraraut/mobotix-move:1.23.3.2`

:point_right: If you have skipped the step [3. ECR Configs and Docs](https://hackmd.io/iKUbbfZ7RYGbXT_tnBd1vg?both#3-ECR-Configs-and-Docs), do it before submitting to the ECR. Ensure that your `ecr-meta/ecr-science-description.md` and `sage.yaml` files are properly configured for this process.

#### Versioning your code
:exclamation: You can not resubmit the plugin to ECR with the same version number again.
- So think about how you change it every time you resubmit to ERC and make your style of versioning.
- I use `v0.y.m.d` e.g. `v0.23.3.4` but then I can only have 1 version a day, so now I am thinking of adding increamemtal integer to it (I am not sure if that's a great idea :thinking_face:) 

### After ECR registry test (generally not required)
1. Generally successfully tested plugins just works. However, in case they are failing in the scheduled jobs after running for a while or after sucessfully running in the above tests, do the following.
2. To test a plugin on a node after it has been built on the ECR, follow these steps: 
```bash
sudo pluginctl run --name test-motion registry.sagecontinuum.org/bhupendraraut/cloud-motion:1.23.01.24 -- -input top
```
2. This command will execute the plugin with the specified ECR image (version 1.23.01.24), passing the "-input top" argument to the plugin (Note `--` after the image telling pluginctl that these argiments are for plugin).

:point_right: Note the use of `sudo` in all commands on the node.

Assuming that the plugin has been installed correctly and the ECR image is available, running this command should test the "test-motion" plugin on the node. 

You may also have to call the `kubectl` <<POD>> commands as in testing section if this fails.

## Scheduling (Incomplete)
:exclamation:If you get an error like `registry not found` or something similar. Ask to update registry on our slack channel.
    
- Follow this [link](https://docs.waggle-edge.ai/docs/tutorials/schedule-jobs) to get an understanding of how to submit a job
- Here are the parameters we set for the mobotix sampler plugin,

```less=
-name thermalimaging registry.sagecontinuum.org/bhupendraraut/mobotix-sampler:1.22.4.13 \
   --ip 10.31.81.14 \
   -u admin \
   -p wagglesage \
   --frames 1 \
   --timeout 5 \
   --loopsleep 60
```
- Your science rule can be a cronjob (More information can be found here [https://github.com/waggle-sensor/sciencerule-checker/blob/master/docs/supported_functions.md#cronjobprogram_name-cronjob_time]()
- This runs every 15 minutes
`"thermalimaging": cronjob("thermalimaging", "*/15 * * * *")`. Use [Crontab Guru](https://crontab.guru/).
- You can also make it triggered by a value (Please read https://github.com/waggle-sensor/sciencerule-checker/blob/master/docs/supported_functions.md for supported functions).

    
NOTE: ==Getting camera ip==
`nmap -sP <10.31.81.1/24>`
cameras are normally in 10.31.81.1X range. Does not work if you have user account.

NOTE: ==Getting serial device ip==.    
    
### Scheduling Script
:sparkles: Check user friendly [job submission UI](https://portal.sagecontinuum.org/jobs/all-jobs) which is mostly  stable now (March 2023)

:green_book: Check [sesctl docs](https://github.com/waggle-sensor/edge-scheduler/tree/main/docs/sesctl) for command line tool.

1. :point_up: Do not use capital letters/dots in the job name. (someone please explain the rules to me)
2. :point_up: make the plugin `public` in sage app portal.
3. Also if you wanna run your plugins not all at the same time. use this example.

```less=
name: w030-k3s-upgrade-test
plugins:
- name: object-counter-bottom
  pluginSpec:
    image: registry.sagecontinuum.org/yonghokim/object-counter:0.5.1
    args:
    - -stream
    - bottom_camera
    - -all-objects
    selector:
      resource.gpu: "true"
- name: cloud-cover-bottom
  pluginSpec:
    image: registry.sagecontinuum.org/seonghapark/cloud-cover:0.1.3
    args:
    - -stream
    - bottom_camera
    selector:
      resource.gpu: "true"
- name: surfacewater-classifier
  pluginSpec:
    image: registry.sagecontinuum.org/seonghapark/surface_water_classifier:0.0.1
    args:
    - -stream
    - bottom_camera
    - -model
    - /app/model.pth
- name: avian-diversity-monitoring
  pluginSpec:
    image: registry.sagecontinuum.org/dariodematties/avian-diversity-monitoring:0.2.5
    args:
    - --num_rec
    - "1"
    - --silence_int
    - "1"
    - --sound_int
    - "20"
- name: cloud-motion-v1
  pluginSpec:
    image: registry.sagecontinuum.org/bhupendraraut/cloud-motion:1.23.02.20
    args:
    - --input
    - bottom_camera
- name: imagesampler-bottom
  pluginSpec:
    image: registry.sagecontinuum.org/theone/imagesampler:0.3.1
    args:
    - -stream
    - bottom_camera
- name: audio-sampler
  pluginSpec:
    image: registry.sagecontinuum.org/seanshahkarami/audio-sampler:0.4.1
nodeTags: []
nodes:
  W030: true
scienceRules:
- 'schedule(object-counter-bottom): cronjob("object-counter-bottom", "*/5 * * * *")'
- 'schedule(cloud-cover-bottom): cronjob("cloud-cover-bottom", "01-59/5 * * * *")'
- 'schedule(surfacewater-classifier): cronjob("surfacewater-classifier", "02-59/5
  * * * *")'
- 'schedule("avian-diversity-monitoring"): cronjob("avian-diversity-monitoring", "*
  * * * *")'
- 'schedule("cloud-motion-v1"): cronjob("cloud-motion-v1", "03-59/5 * * * *")'
- 'schedule(imagesampler-bottom): cronjob("imagesampler-bottom", "04-59/5 * * * *")'
- 'schedule(audio-sampler): cronjob("audio-sampler", "*/5 * * * *")'
successCriteria:
- Walltime(1day)

```

here
objecct-counter runs at 0, 5, 10, etc
cloud-cover: 1, 6, 11, etc.
surface water: 2, 7, 12, etc.
cloud-motion: 3, 8, 13, etc.
image-sampl: 4, 9, 14, etc.


#### My script for cmv

```less=
name: cloud-motion-v1
plugins:
- name: cloud-motion-v1
  pluginSpec:
    image: registry.sagecontinuum.org/bhupendraraut/cloud-motion:1.23.01.24
    args:
    - --input
    - 'top'
nodeTags: []
nodes:
  W023: null
  W02B: null
  W02C: null
  W06D: null
  W06F: null
  W07A: null
  W07B: null
  W07D: null
  W07E: null
  W07F: null
  W015: null
  W019: null
  W021: null
  W024: null
  W029: null
  W039: null
  W078: null
  W079: null
  W080: null
  W083: null
  W084: null
  W088: null
scienceRules:
- 'schedule("cloud-motion-v1"): cronjob("cloud-motion-v1", "* 12-23 * * *")'
successCriteria:
- WallClock('1day')
```


### Debugging Failed Jobs
Do you know how to identify why a job is failing using this code? By specifying the plugin name and node, the code will print out the reasons for job failure within the last 60 minutes.

```python=
from utils import *

mynode = "w030"

myplugin = "water"
df = fill_completion_failure(parse_events(get_data(mynode, start="-60m")))
for _, p in df[df["plugin_name"].str.contains(myplugin)].iterrows():
    print(p["error_log"])
```

https://github.com/waggle-sensor/edge-scheduler/blob/main/scripts/analysis/utils.py



## Downloading the data
    
[Sage docs for accessing-data](https://docs.waggle-edge.ai/docs/tutorials/accessing-data)
    
    
### See Your Data on Sage Portal
To check your data on Sage Portal, follow these steps:
1. Click on the Data tab at the top of the portal page.
2. Select Data Query Browser from the dropdown menu.
3. Then, select your app in the filter. 
This will show all the data that is uploaded by your app using the `plugin.publish()` and `plugin.upload()` methods.

In addition, you can data visualize it as a time series, select multiple variables to visualize together in a chart, which can be useful for identifying trends or patterns. 

![](https://i.imgur.com/5W9jAfw.png)


### Download all images with wget
1. Visit https://training-data.sagecontinuum.org/
2. select node is and period for data.
3. Select required data and download text file _urls-xxxxxxx.txt_ with urls
4. To select only the top camera images, use `vim` command :`g/^\(.*top\)\@!.*$/d`. This will delete urls that do not contain the word 'top'
5. Copy the following command from the website and run in your terminal. `wget -r -N -i urls-xxxxxxx.txt` 


### Sage data client for text data
- Sage data client https://github.com/sagecontinuum/sage-data-client/blob/main/examples/plotting_example.ipynb
- pypi link https://pypi.org/project/sage-data-client/
`pip install sage-data-client`
    
:green_book:Documnetation for [accesing the data.](https://docs.sagecontinuum.org/docs/tutorials/accessing-data)



### Querying Data Example

The `sage_data_client` provides `query()` function ehich takes the parameters:

`start`: The start time of the query as a string.
`end`: The end time of the query as a string.
`filter`: A dictionary with key-value pairs to apply filter. The function returns a pandas DataFrame object.

```python=
import sage_data_client
import urllib
import tempfile
from IPython.display import Image
import imageio as io
import os

from PIL import Image
from PIL import ImageFont
from PIL import ImageDraw 

df = sage_data_client.query(
    start="2023-02-10T12:00:00Z",
    end="2023-02-22T23:00:00.000Z", 
    filter={
        "plugin": "registry.sagecontinuum.org/theone/imagesampler.*",
        "vsn": "W084",
        "job": "imagesampler-top"
    }
).sort_values('timestamp')

with io.get_writer('taft.mp4', fps=5) as writer:
    for i in df.index:
        uurl = df.value[i]
        timestamp = df.timestamp[i]
        try:
            tempf = tempfile.NamedTemporaryFile()
            urllib.request.urlretrieve(uurl, tempf.name)
            
            img = Image.open(tempf.name)
            draw = ImageDraw.Draw(img)
            font = ImageFont.truetype("SFNS.ttf", 64)
            draw.text((800, 1800),timestamp.strftime('TAFT Waggle Node 84 %y-%m-%d %H:%M Z'),(255,255,255),font=font)
            img.save(tempf.name + '.jpg')
            
            image = io.imread(tempf.name + '.jpg')
            writer.append_data(image)
        except urllib.error.HTTPError:
            print('Image skipped ' + uurl)
            
writer.close()
```



