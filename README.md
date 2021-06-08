# ios-signer-unraid

This is a free and simple builder for [ios-signer-service](https://github.com/SignTools/ios-signer-service). It uses a Continous Integration (CI) provider to pull, sign, and upload any iOS apps to your `ios-signer-service`.

The following providers are supported:

- [GitHub Actions](https://docs.github.com/en/actions)
- [Semaphore CI](https://semaphoreci.com/)

You only need to configure one provider.

> ### :warning: Developer accounts are only supported on GitHub Actions for now!

## Setup

1. Create a GitHub account
2. Click on the `Use this template` button at the top of this page
3. Give the new project a name and set the visibility to `Private`
4. Create the new project

Alternatively, you can also manually clone this repo into a new private repository.

You now need to configure a CI provider. You only need one of them:

### GitHub Actions

1. [Open](https://github.com/settings/tokens/new) the Personal access token generation page
2. Select (grant) the `workflow` scope
3. Generate the token
4. Copy the token and put it in a safe place (notpad etc)

This is the token you need for your `ios-signer-service`'s builder configuration.

### Generate a Apple Dev Cerificate

The certificate is a file with an extension `.p12`. To obtain it, follow the instructions below:

**On macOS:** Install [Xcode](https://developer.apple.com/xcode/) and open the `Account Preferences` (A). Sign into your account using the plus button. Select your account and click on `Manage Certificates...`. In the new window (B), click the plus button and then `Apple Development`. Click `Done`. Now open the `Keychain` app (C). There you will find your certificate and private key. Select them by holding `Command`, then right-click and select `Export 2 items...`. This will export you the `.p12` file you need.

<table>
<tr>
    <th>A</th>
    <th>B</th>
    <th>C</th>
</tr>
<tr>
    <td><img src="https://raw.githubusercontent.com/SignTools/ios-signer-service/master/img/6.png"/></td>
    <td><img src="https://raw.githubusercontent.com/SignTools/ios-signer-service/master/img/7.png"/></td>
    <td><img src="https://raw.githubusercontent.com/SignTools/ios-signer-service/master/img/5.png"/></td>
</tr>
</table>

### Create a provisioning profile on apple dev portal

You will need to prepare a signing profile for use with the signing service.

- **Certificate + provisioning profile**

  If you have a paid developer account, it is highly recommended to use this method. Doing so will save you from a lot of limitations. To get a provisioning profile (`.mobileprovision` file), [create one](https://developer.apple.com/library/archive/recipes/ProvisioningPortal_Recipes/CreatingaDevelopmentProvisioningProfile/CreatingaDevelopmentProvisioningProfile.html) from your developer portal and download it. You will probably want it to be a `Development` type and not `Distribution`, so that you can have a `wildcard` application identifier and app debugging entitlement (`get-task-allow`). For the differences, check the [FAQ](FAQ.md) page. Also don't forget to [register the UDID](https://developer.apple.com/library/archive/recipes/ProvisioningPortal_Recipes/AddingaDeviceIDtoYourDevelopmentTeam/AddingaDeviceIDtoYourDevelopmentTeam.html#//apple_ref/doc/uid/TP40011211-CH1-SW1) of each device that you want to sideload to. Read ahead on how to get your certificate.

- **Create an App Identifier**

1. Go to the apple dev portal and click [+](https://i.imgur.com/c2mCGPD.png) on identifiers
2. Register it as an APP IDs
3. Select a type: App
4. Name the app ID whatever you want. This will be your wild card app ID. 
5. Select [Wildcard](https://i.imgur.com/IJdyGqT.png) and name the bundle ID whatever you want with an asterisks after it. 
6. Register the APP ID

- **Add a UDID's to Your Dev Portal**

1. Add your device(s) to your dev [portal](https://i.imgur.com/iTaO3xr.png)
2. IMPORTANT: ADD ALL THE DEVICES TO YOUR PORTAL BEFORE SIGNING WITH AND EXPORTING PROFILE. YOU WILL NEED RECREATE YOUR PROFILE AND RESIGN ANY APPS IF YOU WANT TO ADD MORE DEVICES DOWN THE LINE. 

- **Create of Provisioning Profile**

1. Click the [+](https://i.imgur.com/w3YsEtW.png)
2. Register with a ios [Dev](https://i.imgur.com/bDHIZJI.png) profile
3. Select your newely created App ID
4. Select your Development certificate you created in xcode
5. Select all devices you added in the previous step
6. Save and then download your profile

### Webserver Configuration (Self Hosting on Unraid)

- **This section will pertain to self hosting the webserver via docker on unraid.**

1. Pull the docker image from CA called "ios signer service"
2. Keep all default mappings. If you need to change the host port you can do so now. 
3. Create a yml file called signer-cfg.yml
4. Set github "`enable`" to: true
5. Change the repo name to whatever you named your private repo clone in the steps above
6. Change your "`org_name`" to your github username
7. Paste your API github token in the "`token:`" field
8. Change the "`Server Url`" to your subdomain you made for this webserver. I use signer.domain.tld.
9. You need to pass through authentication to this webserver so you can secure it. I use authelia. The webserver has built in basic web auth. I wouldn't recommend using that for web facing apps.

- **This section will pertain to Authelia config for this Webserver**

 > :warning: **Your authelia config should be set up how you want it. This will be an example on how to set it up JUST FOR IOS SIGNER! I will not HELP with authelia setup. There are plenty of guides on how to do so.**  

1. I use wild card authelia auth for all of my services I host.
2. Your first rule should be your apps and jobs bypass. You can copy that rule down below. 
3. You then need to enable a policy for securing the webserver. 
4. I use a group named "`ios`" and then have all of my friends/family that use the ios signer a part of that group. They will need their own login (Or you can use just one for them just make the password strong :] )
5. IN your user_database.yml you need to grant those friends and family memebers the group "`ios`". Here is an [example](https://i.imgur.com/ZUSCKUF.png). You should also grant your self that ios group as well as your admin group. 
5. Your admin and wildcard rule should be below these two rules. 
6. Below is how I have my authelia config set up under access control section. 

```yml
access_control:
  # Default policy can either be 'bypass', 'one_factor', 'two_factor' or 'deny'.
  # It is the policy applied to any resource if there is no policy to be applied
  # to the user.
  default_policy: deny
  
  rules:
    # Rules applied to 'admins' group
    - domain: sign.domain.tld
      resources:
      - "^/apps/.*$"
      - "^/jobs/.*$"
      - "^/jobs$"
      policy: bypass
    - domain: sign.domain.tld
      subject:
        - "group:ios"
      policy: one_factor
    - domain: "*.domain.tld"
      subject:
        - "group:admins"
      policy: two_factor
```

**Default config for signer-cfg.yml:**

> :warning: **Don't forget to set "`enable: true`" on the builder that you are configuring!**

```yml
# here you define the builder you created in the previous section
# configure only the one that matches yours
builder:
  # GitHub Actions
  github:
    enable: false
    # the name you gave your builder repository
    repo_name: ios-signer-ci
    # your GitHub profile/organization name
    org_name: YOUR_ORG_NAME
    workflow_file_name: sign.yml
    # your GitHub personal access token that you created with the builder
    token: YOUR_GITHUB_TOKEN
    ref: master
  # Semaphore CI
  semaphore:
    enable: false
    # the name you gave to your Semaphore CI project
    project_name: YOUR_PROJECT_NAME
    # your Semaphore CI profile/organization name
    org_name: YOUR_ORG_NAME
    # your Semaphore CI token that you got when creating the builder
    token: YOUR_SEMAPHORE_TOKEN
    ref: refs/heads/master
    secret_name: ios-signer
  # your own self-hosted Mac builder
  selfhosted:
    enable: false
    # the url of your builder
    url: http://192.168.1.133:8090
    # the auth key you used when you started the builder
    key: SOME_SECRET_KEY
# the base url used by the builder to reach this server
# leave empty if using a tunnel provider, it will set this automatically
server_url: https://mywebsite.com
# whether to redirect all http requests to https
redirect_https: false
# where to save data like apps and signing profiles
save_dir: data
# apps older than this time will be deleted when a cleanup job is run
cleanup_mins: 10080
# how often does the cleanup job run
cleanup_interval_mins: 5
# apps that have been processing for more than this time will be marked as failed
sign_timeout_mins: 15
# this protects the web ui with a username and password
# definitely enable it if you are using a tunnel provider
basic_auth:
  enable: false
  username: "admin"
  # don't forget to change the password
  password: "admin"
```
- **Placing files in your folders**

> :warning: **You need to match the files names exactly as they are shown below. For an example, your certificate must be named exactly `cert.p12`. Be aware that Windows may hide the extensions by default.**

- **Certificate + provisioning profile**

  ```appdata folder
  data (ios folder)
  |____profiles
  | |____my_profile                # any unique string that you want
  | | |____cert.p12                # the signing certificate
  | | |____cert_pass.txt           # the signing certificate's password
  | | |____name.txt                # a name to show in the web interface
  | | |____prov.mobileprovision    # the signing provisioning profile
  | |____my_other_profile
  | | |____...
  ```

1. The appdata folder will have an extra "data" folder inside it. Disregard that folder. Use the profiles folder [here:](https://i.imgur.com/m9noew2.png)
2. Open the profiles profile and create a folder in there and name it whatever you want. I use "`my_name`". 
3. Place all of your files in your newly created folder [files](https://i.imgur.com/P9RhKkL.png). Everything must have the names above. 
4. The .mobileprovision files name can be whatever you named your profile. 
5. Upload your signer-cfg.yml file to the config folder. 

### Start the container and sign an App

1. Hopefully you followed along and my instructions were good. Your app should now upload and sign with github actions. 
2. If you have any issues open up an issue and I can reach out to you if time persists. 
> :warning: **If you launch this on anything but unraid "`DO NOT`" open an issue. I probably can't help you.**
