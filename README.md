# vibium-api-docs

OpenAPI documentation is available at `https://api.beta.multiviz.com/docs`

## Use case: get modes for sensor data

In this example we will analyze behaviour of a sensor that submits waveform data.

1. Create a source.

```
curl -X 'POST' \
  'https://api.beta.multiviz.com/sources/' \
  -H 'accept: application/json' \
  -H 'X-Vibium-Api-Key: <your_api_token>' \
  -H 'Content-Type: application/json' \
  -d '{
  "source_id": "REYK_3_FAN_3__NDE_H_MOTOR_3__1Khz",
  "meta": {
    "location": "REYK 3",
    "assetName": "FAN 3 (REYK 3)",
    "sensorName": "NDE H MOTOR 3",
    "measurementName": "1k Acc X"
  },
  "channels": [
    "acc_x"
  ]
}'
```

You can check which sources you have created by calling:

```
curl -X 'GET' \
  'https://api.beta.multiviz.com/sources/' \
  -H 'accept: application/json' \
  -H 'X-Vibium-Api-Key: <your_api_token>'
```

2. Upload measurement for the source.

```
curl -X 'POST' \
  'https://api.beta.multiviz.com/sources/REYK_3_FAN_3__NDE_H_MOTOR_3__1Khz/measurements' \
  -H 'accept: application/json' \
  -H 'X-Vibium-Api-Key: <your_api_token>' \
  -H 'Content-Type: application/json' \
  -d '[
  {
    "duration": 0.8,
    "data": {
      "acc_x": [
        0.000539502827450633,
        0.0010921749053522944,
        0.0006084455526433885,
        0.000581291678827256,
        0.0006375821540132165
        ... more data ...
      ]
    },
    "timestamp": 1695582344199
  }
]'
```

Plese note that under `data` field you upload measurements for the corresponding channel name `acc_x` that you specified in step 1. The duration is specified in seconds. The timestamp is specified in milliseconds from epoch.

Repeat step 2 for every measurements that you want to submit for the source. We need at least 20 to run the analysis.

3. Analyze source behaviour.

```
curl -X 'POST' \
  'https://api.beta.multiviz.com/analyses/requests' \
  -H 'accept: application/json' \
  -H 'X-Vibium-Api-Key: <your_api_token>' \
  -H 'Content-Type: application/json' \
  -d '{
  "feature": "ModeId",
  "params": {},
  "source_id": "REYK_3_FAN_3__NDE_H_MOTOR_3__1Khz"
}'
```

It will return a `request_id` parameter like:

```
{
  "request_status": "queued",
  "request_id": "6d753898c06b4096c3d27bd5dede553e"
}
```

You can use this `request_id` to track analysis status like this:

```
curl -X 'GET' \
  'https://api.beta.multiviz.com/analyses/requests/6d753898c06b4096c3d27bd5dede553e' \
  -H 'accept: application/json' \
  -H 'X-Vibium-Api-Key: <your_api_token>'
```

It will provide you current analysis status. Once it is `successful` you can request analysis results:

```
curl -X 'GET' \
  'https://api.beta.multiviz.com/analyses/requests/6d753898c06b4096c3d27bd5dede553e/results' \
  -H 'accept: application/json' \
  -H 'X-Vibium-Api-Key: <your_api_token>'
```

Sometimes analysis might fail for some reason. You will see it in the result in this case. Please contact us in case analysis keeps failing for your source.

4. Upload new measurements.

When you need to upload new measurements for the source simply repeat steps 2 and 3.

NOTE: For a triaxial sensor, each axis is to be treated as a separate source, and steps 1 to 3 directly apply.
