---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python
  - shell


search: true
---

# Introduction

Documentation for the Corona.ai Prediction API. This API is currently in **ALPHA**.

# Client access token retrieval

```python
import requests
data = requests.post(
    "https://eggwhite.eu.auth0.com/oauth/token",
    headers={"Content-Type": "application/json"},
    data={
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

Retrieving the access token for interaction with our API can be achieved by sending a request to the
Auth0 authentication servers. Tokens have a lifespan of 24 hours. You need to re-acquire a new one
before the old one runs out in order to keep using the API without interruption.

We recommend the `requests` library for Python for ease of use.

# API Usage

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
  --url http://35.195.7.94:9998/transcripts/ENST00000622645/predictions/120/Ala \
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
    "residue_type": "Met",
    "residue_number": 1,
    "variant_type": "Ala",
    "source_max": 0.2405140951,
    "source_min": 0.0634767117,
    "spread": 0.1770373834,
    "transcript_id": "ENST00000622645",
    "deleterious": 0.0903938699
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


# Errors

Currently, all errors (including authentication errors) will yield a `500` response and a JSON body,
containing `{"Error": "Internal Server Error"}`.
