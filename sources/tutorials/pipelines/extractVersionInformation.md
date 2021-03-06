page_title: Using integrations with your runSh job
page_description: Continuous deployment tutorials
page_keywords: containers, lxc, docker, Continuous Integration, Continuous Deployment, CI/CD, testing, automation, Tokens, account settings

#Extract resource version information
The `runSh` job type is a custom job that lets you run custom scripts as part of your deployment pipeline. This tutorial is only relevant to jobs of type `runSh` since managed jobs automatically handle the scenario described on this page.

If you have a custom job in your deployment pipeline on Shippable which has an `IN` resource, your custom job is triggered each time that resource changes. You might need to extract some information from the resource in your custom job's scripts. This tutorial explains how to do that. 

Let us assume you have a custom job `myCustomJob` that takes an input resource `myImage` which is a Docker image generated by the previous job in the pipeline, `someJob`. Each time `someJob` pushes a new tag for `myImage`, Shippable will update the version of the image resource and trigger anything that depends on `myImage`, which in this case is your custom job `myCustomJob`.  

<img src="../../images/pipelines/extractVersionInformation.png" alt="Extracting information from a version of a resource" style="width:500px;"/> 
<br>

Let us define myImage resources in `shippable.resources.yml`:

```
resources:  
  #define image resource
  - name: myImage							#required
    type: image								#required
    integration: myIntegration				#required
    pointer:
      sourceName: "myRepo/myImage"			#required
    seed:
      versionName: 1.1						#required

```


The custom job will be defined in `shippable.jobs.yml` as shown below. myImage is an `IN` value and the job runs a script `doSomething.sh`:


```
jobs:

  - name: myCustomJob
    type: runSh
    steps:
      - IN: myImage
      - TASK:
        - script: ./IN/mexec-repo/gitRepo/doSomething.sh

```

Now, you want to access information from the input resource myImage in your doSomething.sh. In order to do this, you need to know that all version specific information for a resource is stored in a file `version.json` stored in the folder `./IN/<resource name>`. The contents of `version.json` depend on the type of resource.

You can access anything from this file as long as you know what you need. In our example, the resource is an image. The `version.json` file looks like this for an image resource:

```
{
  "operation": "IN",
  "resourceId": 135,
  "name": "dv-image",
  "sourceName": "library/nginx",
  "projectId": "57b69cb8c9e2c91700bb8fb2",
  "isConsistent": true,
  "type": "image",
  "propertyBag": {
    "yml": {
      "name": "dv-image",
      "type": "i mage",
      "integration": "deepika-docker",
      "pointer": {
        "sourceName": "library/nginx"
      },
      "seed": {
        "versionName": "latest"
      }
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 126,
    "versionNumber": 1,
    "versionName": "latest",
    "propertyBag": {
      
    }
  },
  "subscriptionIntegrationId": 3
}

```

Let us assume you want to access the value of the versionName field in your custom script `doSomething.sh`. Here is how you would do it:

```
findVersionName() {
  export versionName=$(cat ./IN/myImage/version.json | jq -r '.version.versionName')
}

```
This will export the `versionName` field from `version.json` in an environment variable `$versionName`.

Please note that we're using [`jq`](https://stedolan.github.io/jq/), a command line json processing tool for the example above. To use this tool, you'll need to install it in your custom job in `shippable.jobs.yml` before running your custom script:

```
jobs:

  - name: myCustomJob
    type: runSh
    steps:
      - IN: myImage
      - TASK:
        - script: sudo apt-get install -y jq
        - script: ./IN/mexec-repo/gitRepo/doSomething.sh

```

##version.json reference

The example above shows the `version.json` for an `image` resource. To use this example with other types of resources, look up the json below.


###gitRepo

`version.json` for a gitRepo resource is:

```
{
  "operation": "IN",
  "resourceId": 19156,
  "name": "infra-repo",
  "sourceName": "ric03uec/infra",
  "projectId": "57a8bf2e61b41c0f00c7378c",
  "isConsistent": true,
  "type": "gitRepo",
  "propertyBag": {
    "yml": {
      "name": "infra-repo",
      "type": "gitRepo",
      "integration": "ric03uec-github",
      "pointer": {
        "sourceName": "ric03uec/infra",
        "branch": "master"
      }
    },
    "sysIntegrationName": "ric03uec-github",
    "sysDeployKey": {
      "public": "public key",
      "private": "private key"
    },
    "sysPassword": "VniMvjaRgJuiLBlS",
    "sysUserName": "infra",
    "sysWebhookExternalId": 9430397,
    "normalizedRepo": {
      "name": "infra",
      "fullName": "ric03uec/infra",
      "repositoryProvider": "github",
      "repositoryUrl": "https://api.github.com/repos/ric03uec/infra",
      "sourceId": 65223724,
      "sourceDescription": "infra provisioning repo",
      "isPrivateRepository": false,
      "isFork": false,
      "repositorySshUrl": "git@github.com:ric03uec/infra.git",
      "repositoryHttpsUrl": "https://github.com/ric03uec/infra.git",
      "language": "Shell",
      "sourceForksCount": 0,
      "sourceStargazersCount": 0,
      "sourceWatchersCount": 0,
      "sourceSize": null,
      "sourceDefaultBranch": "master",
      "sourcePushed": "2016-08-08T17:14:35Z",
      "sourceCreated": "2016-08-08T17:13:44Z",
      "sourceUpdated": "2016-08-08T17:14:35Z",
      "sourceRepoOwner": {
        "login": "ric03uec",
        "id": 1110988,
        "avatar_url": "https://avatars.githubusercontent.com/u/1110988?v=3",
        "gravatar_id": "",
        "url": "https://api.github.com/users/ric03uec",
        "html_url": "https://github.com/ric03uec",
        "followers_url": "https://api.github.com/users/ric03uec/followers",
        "following_url": "https://api.github.com/users/ric03uec/following{/other_user}",
        "gists_url": "https://api.github.com/users/ric03uec/gists{/gist_id}",
        "starred_url": "https://api.github.com/users/ric03uec/starred{/owner}{/repo}",
        "subscriptions_url": "https://api.github.com/users/ric03uec/subscriptions",
        "organizations_url": "https://api.github.com/users/ric03uec/orgs",
        "repos_url": "https://api.github.com/users/ric03uec/repos",
        "events_url": "https://api.github.com/users/ric03uec/events{/privacy}",
        "type": "User",
        "site_admin": false
      },
      "permissions": null,
      "roles": null,
      "repositoryHtmlUrl": null
    }
  },
  "versionDependencyPropertyBag": {},
  "version": {
    "versionId": 88535,
    "versionNumber": 23,
    "versionName": "74943404bb84b6042141f792cfe68853a3dbc50d",
    "propertyBag": {
      "shaData": {
        "providerDomain": "github.com",
        "branchName": "master",
        "isPullRequest": false,
        "pullRequestNumber": null,
        "pullRequestBaseBranch": null,
        "commitSha": "74943404bb84b6042141f792cfe68853a3dbc50d",
        "beforeCommitSha": "f0a61acb0025b2f346da512fb35b361b79100dcd",
        "commitUrl": "https://github.com/ric03uec/infra/commit/74943404bb84b6042141f792cfe68853a3dbc50d",
        "commitMessage": "updting provision script",
        "baseCommitRef": "",
        "headCommitRef": "",
        "headPROrgName": "",
        "compareUrl": "https://github.com/ric03uec/infra/compare/f0a61acb0025b2f346da512fb35b361b79100dcd...74943404bb84b6042141f792cfe68853a3dbc50d",
        "skipDecryption": false,
        "isGitTag": false,
        "gitTagName": null,
        "gitTagMessage": null,
        "isRelease": false,
        "releaseName": null,
        "releaseBody": null,
        "releasedAt": null,
        "isPrerelease": false,
        "committer": {
          "email": "devashish.86@gmail.com",
          "login": "ric03uec",
          "displayName": "Devashish"
        },
        "lastAuthor": {
          "email": "devashish.86@gmail.com",
          "login": "ric03uec",
          "displayName": "Devashish"
        },
        "triggeredBy": {
          "email": "devashish.86@gmail.com",
          "login": "ric03uec"
        }
      }
    }
  },
  "subscriptionIntegrationId": 2232
}

```


###image

`version.json` for a image resource is:

```
{
  "operation": "IN",
  "resourceId": 135,
  "name": "dv-image",
  "sourceName": "library/nginx",
  "projectId": "57b69cb8c9e2c91700bb8fb2",
  "isConsistent": true,
  "type": "image",
  "propertyBag": {
    "yml": {
      "name": "dv-image",
      "type": "i mage",
      "integration": "deepika-docker",
      "pointer": {
        "sourceName": "library/nginx"
      },
      "seed": {
        "versionName": "latest"
      }
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 126,
    "versionNumber": 1,
    "versionName": "latest",
    "propertyBag": {
      
    }
  },
  "subscriptionIntegrationId": 3
}

```


###dockerOptions

`version.json` for a dockerOptions resource is:

```
[
  {
    "name": "docker-options",
    "images": [
      {
        "image": "library/nginx",
        "tag": "latest",
        "params": {
          "MONGO_API_URL": "localhost:28017"
        },
        "dockerOptions": {
          "memory": 64,
          "cpuShares": 256,
          "portMappings": [
            "80:80"
          ]
        },
        "name": "dv-image",
        "versionNumber": 1
      }
    ],
    "replicas": 1
  }
]

```

###params

`version.json` for a params resource is:

```
{
  "operation": "IN",
  "resourceId": 138,
  "name": "dv-params",
  "sourceName": null,
  "projectId": "57b69cb8c9e2c91700bb8fb2",
  "isConsistent": true,
  "type": "params",
  "propertyBag": {
    "yml": {
      "name": "dv-params",
      "type": "params",
      " version": {
        "params": {
          "MONGO_API_URL": "localhost:28017"
        }
      },
      "flags": [
        "dv"
      ]
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 129,
    "versionNumber": 1,
    "versionName": null,
    "propertyBag": {
      "params": {
        "MONGO_AP I_URL": "localhost:28017"
      }
    }
  }
}

```

###replicas

`version.json` for a replicas resource is:

```
{
  "operation": "IN",
  "resourceId": 139,
  "name": "dv-scaler",
  "sourceName": null,
  "projectId": "57b69cb8c9e2c91700bb8fb2",
  "isConsistent": true,
  "type": "replicas",
  "propertyBag": {
    "yml": {
      "name": "dv-scaler",
      "type": "replica s",
      "version": {
        "count": 1
      },
      "flags": [
        "replicas"
      ]
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 127,
    "versionNumber": 1,
    "versionName": null,
    "propertyBag": {
      "count": 1
    }
  }
}

```

###cluster

`version.json` for a cluster resource is:

```
{
  "operation": "IN",
  "resourceId": 143,
  "name": "env-ddc",
  "sourceName": "docker-cloud-node",
  "projectId": "57b69cb8c9e2c91700bb8fb2",
  "isConsistent": true,
  "type": "cluster",
  "propertyBag": {
    "yml": {
      "name": "env-ddc",
      "type ": "cluster",
      "integration": "deepika-ddc",
      "pointer": {
        "sourceName": "docker-cloud-node"
      }
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 133,
    "versionNumber": 1,
    "versionName": null,
    "propertyBag": {
      
    }
  },
  "subscriptionIntegrationId": 4
}

```

###notification

`version.json` for a notification resource is:

```
{
  "operation": "IN",
  "resourceId": 61,
  "name": "slackNotification",
  "sourceName": null,
  "projectId": "57b7e770f848cc1a00a8d0bf",
  "isConsistent": true,
  "type": "notification",
  "propertyBag": {
    "yml": {
      "name": "slackNotification",
      "type": "notification",
      "integration": "deepika-slack",
      "pointer": {
        "recipients": [
          "#deepika-test"
        ]
      }
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 73,
    "versionNumber": 1,
    "versionName": null,
    "propertyBag": {
      
    }
  },
  "subscriptionIntegrationId": 5
}

```


###integration

`version.json` for a integration resource is:

```
{
  "operation": "IN",
  "resourceId": 148,
  "name": "dockerIntegration",
  "sourceName": null,
  "projectId": "57b69cb8c9e2c91700bb8fb2",
  "isConsistent": true,
  "type": "integration",
  "propertyBag": {
    "yml": {
      "name": "dockerIntegration",
      "type": "integration",
      "integration": "deepika-docker"
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 138,
    "versionNumber": 1,
    "versionName": null,
    "propertyBag": {
      
    }
  },
  "subscriptionIntegrationId": 3
}
```

###version

`version.json` for a version resource is:


```
{
  "operation": "IN",
  "resourceId": 59,
  "name": "version-resource",
  "sourceName": null,
  "projectId": "57b7e770f848cc1a00a8d0bf",
  "isConsistent": true,
  "type": "version",
  "propertyBag": {
    "yml": {
      "name": "version-resource",
      "type": "version",
      "seed": {
        "versionName": "0.0.1-beta"
      }
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 68,
    "versionNumber": 1,
    "versionName": "0.0.1-beta",
    "propertyBag": {
      
    }
  }
}

```

###job (any type)
In case the `IN` for your custom job is another job, you can still extract version information from it if needed. Similar to resources, all jobs are also versioned.

The `version.json` for a job is shown below. Please note that the property bag depends on the configuration of the job.

```
{
  "operation": "IN",
  "resourceId": 129,
  "name": "dv-man",
  "sourceName": null,
  "projectId": "57b69cb8c9e2c91700bb8fb2",
  "isConsistent": true,
  "type": "manifest",
  "propertyBag": {
    "yml": {
      "name": "dv-man",
      "type": "manifest",
      "steps": [
        {
          "IN": "dv-image",
          "pull": false,
          "versionName": "latest",
          "versionNumber": 0
        },
        {
          "IN": "dv-params",
          "applyTo": [
            "dv-image"
          ]
        },
        {
          "IN": "dv-opts"
        },
        {
          "IN": "trigger-dv-man"
        }
      ],
      "on_success": [
        {
          "NOTIFY": "slackNotification"
        }
      ],
      "on_failure": [
        {
          "NOTIFY": "slackNotification"
        }
      ],
      "flags": [
        "dv"
      ]
    }
  },
  "versionDependencyPropertyBag": {
    
  },
  "version": {
    "versionId": 271,
    "versionNumber": 7,
    "versionName": null,
    "propertyBag": {
      "sha": "933620525107919e5f31148dd42f82419ed361a8"
    }
  }
}

```

