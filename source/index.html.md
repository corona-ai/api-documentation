---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python
  - shell


search: true
---

# Introduction

Documentation for the Corona.ai Prediction API. This API is currently in **ALPHA**.

The API is secured and you need credentials to be able to use it. We use the industry-standard OAuth2 protocol.
For those of you who'd like more information about the ins and outs of OAuth2 please take a look [here](https://auth0.com/docs/protocols/oauth2).
Note that using the API is incredibly easy; you don't really need an in-depth understanding of OAuth2 to be able to use this API.
This documentation contains all the information you need to get started!   

# Client access token retrieval

```python
import requests
data = requests.post(
    "https://eggwhite.eu.auth0.com/oauth/token",
    headers={"Content-Type": "application/json"},
    json={
        "client_id":"<client_id>",
        "client_secret":"<client_secret>",
        "audience":"predictions.ai.bio-prodict.nl",
        "grant_type":"client_credentials"
    }
)
```

```shell
curl --request POST \
  --url https://eggwhite.eu.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"<client_id>","":<client_secret>"audience":"predictions.ai.bio-prodict.nl","grant_type":"client_credentials"}'
```

> Returns the following JSON response:

```json
{ "access_token": "<access_token>", "expires_in": 86400, "token_type": "Bearer" }
```

> Errors

```json
# When your client id is invalid
{ "error": "access_denied", "error_description": "Unauthorized" }
```

We use the Auth0.com service to manage authentication and authorization.
Please check that you have received a _client_id_ and a _client_secret_ before continuing.
Retrieval of your access token is a two-step process:

1. Your application authenticates with Auth0 using its _client_id_ and _client_secret_.

2. Auth0 validates this information and returns an access_token.

Retrieving the access token for interaction with our API can be achieved by sending a request to the
Auth0 authentication servers. Tokens have a lifespan of 24 hours. You need to re-acquire a new one
before the old one runs out in order to keep using the API without interruption.


We recommend the `requests` library for Python for ease of use.

# API Usage

Having an _access_token_, you are now ready to use the API.

## Transcripts overview

```python
transcripts = requests.get(
    "http://35.195.7.94:9998/transcripts",
    headers={"Authorization": "Bearer <access_token>"}
)
```

```shell
curl --request GET \
  --url http://35.195.7.94:9998/transcripts \
  --header 'Authorization: Bearer <access_token>'
```

> Returns the following JSON response:

```
["ENST00000257700", "ENST00000259008", "ENST00000260947", "ENST00000261584", ... ]
```

Returns a full list of the transcripts available on our API. Each of these can be used in other
calls. Transcripts are formatted as Ensemble identifiers. This list is retrieved from the `/transcripts`
endpoint.


## Retrieving variant predictions

> All the request parameters are contained in the URL

```python
variant_data = requests.get(
    "http://35.195.7.94:9998/transcripts/ENST00000622645/predictions/120/Ala",
    headers={"Authorization": "Bearer <access_token>"}
)
```

```shell
curl --request GET \
  --url http://35.195.7.94:9998/transcripts/ENST00000622645/predictions/110/Arg \
  --header 'Authorization: Bearer <access_token>'
```

A single endpoint for variant prediction retrieval exists: `/transcripts/<transcriptId>/predictions/<residueNumber>/<variantType>`.
Using this endpoint requires the following:

- `transcriptId`: A transcript identifier, like those found in the `/transcripts` endpoint response.
- `residueNumber`: A number indicating the residue in this transcript. Residue numbers start at 1.
- `variantType`: The variant you would like to find the information for. Only 3 letter codes are accepted. Case insensitive.

> Returns the following JSON response:

```json
{
    "residue_type": "Arg",
    "residue_number": 110,
    "variant_type": "Arg",
    "source_max": 0.0863609362,
    "source_min": 0.059299522,
    "spread": 0.0270614143,
    "transcript_id": "ENST00000622645",
    "deleterious": 0.0814552796,
    "effect_summary": "0.0815 (benign, high confidence)"
}
```


Identifier | Description
----|------
residue_type | Wildtype amino acid
residue_number | 1-based protein sequence number
variant_type | Variant amino acid type
source_max | Highest sub-predictor prediction
source_min | Lowest sub-predictor prediction
spread | Difference between highest and lowest sub-predictions. Can be interpreted as a measure of uncertainty.
transcript_id | Ensembl transcript id
deleterious | Value indicating deleteriousness. Value between 0 and 1, where a value between 0 and 0.5 means benign and a value between 0.5 and 1 means deleterious.
effect_summary | A simple summary of the prediction with assesment of the impact on the protein and the confidence in the prediction. The first is based on the 'deleterious' score, the second is based on the 'spread' score. Our spread indicates the distance between predictions of our subclassifiers in our ensemble. If the score is below 0.15, we determine that the results of the subclassifiers are concordant, resulting in "high confidence". Between 0.15 and 0.25 indicates "medium confidence" and scores above 0.25 indicate "low confidence".


# Errors

Currently, all errors (including authentication errors) will yield a `500` response and a JSON body,
containing `{"Error": "Internal Server Error"}`.
