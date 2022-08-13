
# 🦅 Webhawk
Machine Learning based web attacks detection.

<p align="center">  
  <img width="600" src="https://images.unsplash.com/photo-1607240376903-9a1f6d09330d?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=2340&q=80">
</p>

## About
Webhawk is an open source machine learning powered Web attack detection tool. It uses your web logs as training data. Webhawk offers a REST API that makes it easy to integrate within your SoC ecosystem. To train a detection model and use it as an extra security level in your organization, follwo the following steps.

## Usage
### Create a settings.conf file
Copy settings_template.conf file to settings.conf and fill it with the required parameters as the following.
```shell
[MODEL]
model:MODELS/the_model_you_will_train.pkl
[FEATURES]
features:length,params_number,return_code,size,upper_cases,lower_cases,special_chars,url_depth
```

### Encode your http logs and save the result into a csv file
```shell
$ python encode.py -a -l ./SAMPLE_DATA/raw-http-logs-samples/aug_sep_oct_2021.log -d ./SAMPLE_DATA/labeled-encoded-data-samples/aug_sep_oct_2021.csv
```
Please note that two already encoded data file are available in ./SAMPLE_DATA/labeled-encoded-data-samples/, in case you would like to move directly to the next step.

### Train a model and test the prediction
Use the http log data from May to July 2021 to train a model, and test it with the data from August to October 2021.
```shell
$ python train.py -a 'dt' -t ./SAMPLE_DATA/labeled-encoded-data-samples/may_jun_jul_2021.csv -v ./SAMPLE_DATA/labeled-encoded-data-samples/aug_sep_oct_2021.csv
```

### Make a prediction for a single log line
```shell
$ python predict.py -m 'MODELS/the_model_you_will_train.pkl' -l '198.72.227.213 - - [16/Dec/2018:00:39:22 -0800] "GET /self.logs/access.log.2016-07-20.gz HTTP/1.1" 404 340 "-" "python-requests/2.18.4"'
```

### REST API
#### Launch the API server
In order to use the API to need first to launch it's server as the following
```shell
$ python3 -m uvicorn api:app --reload --host 0.0.0.0 --port 8000
```
#### Make a predciton request
You can use the following code which based on Python 'requests' (the same in test_api.py) to make a prediction using the REST API
```python
import requests
import json
headers = {
    'accept': 'application/json',
    'Content-Type': 'application/json',
}
data = {
    'http_log_line': '187.167.57.27 - - [15/Dec/2018:03:48:45 -0800] "GET /honeypot/Honeypot%20-%20Howto.pdf HTTP/1.1" 200 1279418 "http://www.secrepo.com/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/61.0.3163.128 Safari/534.24 XiaoMi/MiuiBrowser/9.6.0-Beta"'
}
response = requests.post('http://127.0.0.1:8000/predict', headers=headers, data=json.dumps(data))
print (response.text)
```
It will return the following:
``` python
{"prediction":"0","confidence":"0.9975490196078431","log_line":"187.167.57.27 - - [15/Dec/2018:03:48:45 -0800] \"GET /honeypot/Honeypot%20-%20Howto.pdf HTTP/1.1\" 200 1279418 \"http://www.secrepo.com/\" \"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/534.24 (KHTML, like Gecko) Chrome/61.0.3163.128 Safari/534.24 XiaoMi/MiuiBrowser/9.6.0-Beta\""}
```

### Docker
#### Launch the API server
To launch the prediction server using docker
```shell
$ docker compose build
$ docker compose up
```
## Used sample data
The data you will find in SAMPLE_DATA folder comes from https://www.secrepo.com.

## Documentation
Details on how this tools is build could be found at http://enigmater.blogspot.fr/2017/03/intrusion-detection-based-on-supervised.html

## Todo
To extract/add more features (Eg: day of the time, week, month).
<br>To find a better way to label training data
<br>To add the possibility to use unsupervised learning.

## Contribution
All feedbacks, testing and contribution are very welcome!
If you would like to contribute, fork the project, add your contribution and make a pull request.
