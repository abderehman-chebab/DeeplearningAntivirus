# DeeplearningAntivirus
DeeplearningAntivirus is a malware classification framework, designed primarily for malware detection, it contains a modular toolset for feature extraction, as well as pre-trained models and a ready to use web API for making predictions.


# Use Cases
You will most likely find DeeplearningAntivirus awesome in those cases :

- Extract features from executables and use the dataset for training models or to learn more about them. [Here are instructions to do so](#extractor-standalone)

![architecture-standalone](/assets/malware_revealer_extractor_standalone.png)

- Use the full stack application which covers all the pipeline starting from the feature extraction to the final prediction hosted as a web app. [Here are instructions to do so](#fullstack-app)

![architecture-fullstack](/assets/malware_revealer_fullstack.png)

Your use case don't fall into the one listed above? We will be happy to hear about it.

<a name="extractor-standalone"/>

# How to use the extractor?

It's a simple 3 steps process:

### Chose which features you want to extract

You will need to list the features you wanna extract in a configuration file, check this [wiki page](https://github.com/malware-revealer/malware-revealer/wiki/Make-your-own-extractor-with-your-own-set-of-features) to learn setup the extractor. If one of the features isn't already implemented, you can either make an issue an wait for someone to implement it, or implement it yourself and make a Pull Request :D check this [wiki page](https://github.com/malware-revealer/malware-revealer/wiki/Create-your-own-feature-extractor-class) to learn how to do that.

### Prepare your dataset

Now you need to prepare your dataset, it consists of executable files, organized in subdirectory, each subdirectory represents a class of executables, here is an example:
```bash
    $ tree executables/
    executables/
    ├── 0
    │   └── example.exe
    └── 1
        └── example.exe
```

Here we have represented two classes of executables, either a malware (1) or not (0).

### Extraction

Once you have your configuration file and dataset ready, you can proceed to the extraction. To run the extractor you have two choices, install the requirements and then run it or use the ready to use Docker [image](https://hub.docker.com/r/malwarerevealer/extractor). Here I'm gonna explain the usage of the Docker image since it's simpler.

- If you don't have Docker already installed, [here](https://docs.docker.com/install/) are instructions to do so.
- I will now assume that you have the dataset and the configuration file in the working directory as ./executables/ and ./conf.yaml

```bash
$ docker container run --rm -v $PWD:/data:ro -v $PWD/out:/out malwarerevealer/extractor /data/conf.yaml /data/executables -o /out
```

You will then find the extracted features under ./out

<a name="fullstack-app"/>

# How to use the full stack application?

You can deploy all the components of the application just by running

```bash
$ docker-compose up
```

This will deploy both the predictor and the extractor services as well as an [Nginx](https://www.nginx.com/) server as a reverse proxy for our applications. You can see the architecture [here](#)

This is of course the backend that will host an API for making prediction on executable files.


Here is what actually happens when making predictions (here we used the first version of our CNN model which is actually available at the /cnn/v1/predict endpoint)

![sequence-diagram-predictor](/assets/sequence-diagram-predictor.png)

- The client (web app, CLI utility) start by sending an HTTP POST request with the executable file
- The predictor will use the extractor service to extract features from the executable file (this part is model specific as each model needs a different set of features)
- The predictor will then make inference on the extracted features and return the result to the client


# More details

You can find here the description of each component as well as how we see his future development.

### Feature extractor

It is responsible for taking an executable as input, doing the necessary parsing and extraction of features that should be relevant, to be able to classify our executable as malware or not.

The extractor should be able to extract features from multiple types of executable which are meant to different CPU architectures.

This part will be used to build our dataset from malware samples.

### Model

This is our trained model that should take features from the extractor as input and outputs the likelihood of the executable to be a malware. You can find out more about how we trained our models [here](/notebooks)
