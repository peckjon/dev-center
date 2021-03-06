---
layout: article
title:  "Algorithm Management"
excerpt: "Create, Update, Publish, and Delete Algorithms via the Algorithmia API"
categories: algorithm-development
tags: [algo-dev]
show_related: false
author: jon_peck
redirect_from:
  - /algorithm-development/algorithm-management-api/
image:
  teaser: /icons/algo.svg
---

Using the Algorithmia API, you can create, publish, update, and inspect individual algorithms. Among other things, this allows you to train and deploy your models on Algorithmia completely within your preferred development environment -- without ever interacting with Algorithmia's web-based UI.

### Enabling Algorithm Management

Algorithm management can be enabled when an API key is created, or any time after initial creation, by logging into your account and visiting the [credentials tab of your user profile]({{site.url}}/user#credentials). Ensure that the key you are using has an 'Algorithm Access' value of either:

* **Full Access** - if you'd like to allow the key to call all public and private algorithms you have access to, in addition to managing algorithms you own.

<img src="{{site.cdnurl}}{{site.baseurl}}/images/post_images/algorithm-management/full-access.png" alt="Full Access Algorithm Management Option" class="screenshot">

* **Restricted Access** - if you'd like to fine-tune which algorithms can be called. (Just be sure to select the 'Allow this key to manage my algorithms' checkbox.)

<img src="{{site.cdnurl}}{{site.baseurl}}/images/post_images/algorithm-management/restricted-access.png" alt="Restricted Access Algorithm Management Option" class="screenshot">

To see a full list of specific endpoints a key with this permission can call, see the [Python API for Algorithm Management](https://docs.algorithmia.com/?python#algorithm-management-api). For languages other than Python, an OpenAPI Specification is available via the [unofficial openapi docs](https://documenter.getpostman.com/view/6515899/S1a1aoR6?version=latest#4809ce72-446d-40ef-b857-995e7b2949de) or a [raw openapi spec (under /algorithms)](/v1/openapispec).

### Forward: Python Notebook or CI/CD (via Jenkins or GitHub Actions) for Publishing Algorithms

The step-by-step guide below will familiarize you with the Python and OpenAPI spec for creating Algorithms. However, a complete end-to-end Jupyter Notebook (and Google Colab Notebook) is also available. To see a working end-to-end example of training a Model, then *immediately* deploying it as an Algorithm within the same Notebook, see the [GitHub repo with working Model Deployment Notebooks](https://github.com/algorithmiaio/model-deployment)

In addition, customers using Jenkins or GitHub Actions for their CI/CD should review our [CI/CD Examples for Deploying Algorithms]({{site.baseurl}}/algorithm-development/ci-cd).

For those using another CI/CD tool, or who simply wish to deploy from a simple pure-Python script, the Python scripts used in the GitHub Actions workflow can be used in any Python-capable environment; they are not specific to GitHub Actions. Read through the [documentation](https://github.com/algorithmiaio/model-deployment/tree/master/githubactions_deploy_algorithmia#redeploying-retrained-models) to understand their use, then copy and modify the model_*.py files from that repo and modify them to point to your own Algorithm.

### Step-by-step: Creating and Publishing an Algorithm via the API, using the official Python Client (recommended)

{% if site.enterprise %}
#### Enterprise Users Only: Specifying an On-Premises or Private Cloud Endpoint
If you are running [Algorithmia Enterprise](/enterprise), remember to specify your API endpoint when creating the Python Client:

{% highlight python %}
client = Algorithmia.client('API_KEY', 'https://mylocalendpoint')
{% endhighlight %}

{% endif %}

#### 1. Create your Algorithm

First, set up an Algorithmia client using an API Key from your account, which **must** have "Allow this key to manage my algorithm" enabled.

Then, create your Algorithm by creating a `client.algo('username/algoname')` with the name you wish it to have: 'username' here might be your own username, or the name of an Organization to which you belong, and 'algoname' can be any Algorithm name which doesn't already exist in that userspace.

Next, we call the `.create()` method of that new Algorithm. The fields roughly correspond to those you'd see in the [Add Algorithm]({{site.url}}{{site.baseurl}}/algorithm-development/algorithm-basics/your-first-algo/#create-your-first-algorithm) user interface, but are fully described in the [API Docs](https://docs.algorithmia.com/?python#create-an-algorithm).

{% highlight python %}
import Algorithmia
client=Algorithmia.client('API_KEY')
algo = client.algo('YOUR_USERNAME/Hello')
algo.create(
    details = {
        "label": "Hello World",
    },
    settings = {
        "language": "python3-1",
        "source_visibility": "closed",
        "license": "apl",
        "network_access": "full",
        "pipeline_enabled": True,
        "environment": "cpu"
    }
)
{% endhighlight %}

#### 2. Edit and Build your Algorithm

Now that your Algorithm exists, you can edit the source code the Web IDE, or locally via [Git]({{site.cdnurl}}{{site.baseurl}}/algorithm-development/algorithm-basics/git) (`git clone https://git.algorithmia.com/git/USERNAME/ALGORITHMNAME.git`) or the [CLI]({{site.cdnurl}}{{site.baseurl}}/clients/cli) (`algo clone USERNAME/ALGORITHMNAME`).

Before attempting to publish, you must either click "Build" in the Web IDE, or `git push` your code (which implicitly triggers a build).

#### 3. Publish your Algorithm

Now you can publish your algorithm via a call to `algo.publish`

While an empty dict `{}` will work as the `version_info` parameter, you may also include any of the fields included in the [API Spec](https://docs.algorithmia.com/?python#publish-an-algorithm):

{% highlight python %}
algo.publish(
    version_info = {
        "sample_input": "world"
    }
)
{% endhighlight %}

#### 4. Getting info about your Algorithm

Your Algorithm is now published and runnable. If you wish, you make also take advantage of several other management functions:

* [Update an existing Algorithm](https://docs.algorithmia.com/?python#optional-update-an-algorithm)
* [Force a recompile of your Algorithm](https://docs.algorithmia.com/?python#optional-recompile-your-algorithm)
* [Get info, such as last published version number](https://docs.algorithmia.com/?python#get-info-about-an-an-algorithm)
* [List all versions of your Algorithm & corresponding info](https://docs.algorithmia.com/?python#list-versions-of-an-algorithm)


### ALTERNATIVE: Creating and Publishing an Algorithm via the API, using the OpenAPI Specification

{% if site.enterprise %}
#### Enterprise Users Only: Specifying an On-Premises or Private Cloud Endpoint
If you are running [Algorithmia Enterprise](/enterprise), you must specify a different API endpoint when using these APIs. Simply replace "https://api.algorithmia.com" with the base URL of your own installation.
{% endif %}

#### 1. Create your Algorithm

First, create your algorithm by POSTing to https://api.algorithmia.com/v1/algorithms/USERNAME, where `USERNAME` is your user account, and `sim******` is an API Key from that account with "Allow this key to manage my algorithm" enabled.

Note that the definition of the new Algorithm is in the payload of the request, and the fields roughly correspond to those you'd see in the [Add Algorithm]({{site.url}}{{site.baseurl}}/algorithm-development/algorithm-basics/your-first-algo/#create-your-first-algorithm) user interface:

{% highlight python %}
import requests
url = 'https://api.algorithmia.com/v1/algorithms/USERNAME'
headers = {
  'Authorization': 'sim********',
  'Content-Type': 'application/json'
}
payload = {
    "details": {
        "label": "<string>", #user-readable name of the algorithm
        "summary": "<string>", #markdown describing the Algorithm, for the "docs" tab
        "tagline": "<string>" #one-liner summarizing the Algorithm's purpose
    },
    "name": "<string>", #the short algorithm name (no spaces or special characters)
    "settings": {
        "license": "<string>", #apl, apache2, gpl3, mit
        "network_access": "<string>", #isolated, full
        "pipeline_enabled": <boolean>, #can this algo call other algos?
        "source_visibility": "<string>", #open, closed
        "language": "<string>", #java, javascript, python2-langpack, python3-1, r, ruby, rust, scala
        "environment": "<string>", #cpu, gpu
        "royalty_microcredits": <integer> #0 for none
    },
    "version_info": {
        "sample_input": "<string>" #example input visible to end-user
    }
}
response = requests.request('POST', url, headers = headers, json = payload)
print(response.text)
{% endhighlight %}

#### 2. Edit and Build your Algorithm

Now that your Algorithm exists, you can edit the source code the Web IDE, or locally via [Git]({{site.cdnurl}}{{site.baseurl}}/algorithm-development/algorithm-basics/git) (`git clone https://git.algorithmia.com/git/USERNAME/ALGORITHMNAME.git`) or the [CLI]({{site.cdnurl}}{{site.baseurl}}/clients/cli) (`algo clone USERNAME/ALGORITHMNAME`).

Before attempting to publish, you must either click "Build" in the Web IDE, or `git push` your code (which implicitly triggers a build).

#### 3. Publish your Algorithm

Now you can publish your algorithm via a POST to https://api.algorithmia.com/v1/algorithms/USERNAME/ALGORITHMNAME/versions

While an empty payload dict `{}` will work, you may consider including the payload fields shown below:

{% highlight python %}
import requests
url = 'https://api.algorithmia.com/v1/algorithms/USERNAME/ALGORITHMNAME/versions'
headers = {
  'Authorization': 'sim********',
  'Content-Type': 'application/json'
}
payload = {
    "settings": {
        "algorithm_callability": "<string>" #public, private
    }
    "version_info": {
        "type": "<string>", #default, major, minor, revision
        "semantic_version": "<string>", #if blank, a minor version increment will be used
        "git_hash": "<string>", #if blank, the latest git hash will be used
        "release_notes": "<string>",
        "sample_input": "<string>",
        "sample_output": "<string>"
    }
}
response = requests.request('POST', url, headers = headers, json = payload)
print(response.text)
{% endhighlight %}

You can also include any "details" or "settings" fields as shown in the "Create your Algorithm" step.
