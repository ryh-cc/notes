## Monzo Reponse
Prompted by COPS [request](https://ccycloud.atlassian.net/browse/SEC-1046) to
have an incident response system to handle comms and coordinate all incident
related reporting, status pages etc, this document will look at Monzo Response
with regards to it's deployment and management in Currencycloud.

There is a paid alternative product - Firehydrant - however analysing it is
outside the scope of this document as we want to first assess the suitability
of Response, Monzo's open sourced version.

Response is essentially a Slack bot that handles the communication,
reporting and coordination that needs to happen during and after incidents, allowing 
engineers and stakeholders to concentrate on identifying root cause and solving
the incident.

The bot monitors and reacts to commands as `/incident` to declare an incident
and initiate the process of setting incident lead, creating a dedicated place
(slack channel) to communicate, updating status page etc. More info here:
https://monzo.com/blog/2019/07/08/how-we-respond-to-incidents and
https://github.com/monzo/response.

### Response Components 
Response consists of a Django project using the 
[django-incident-reponse](https://pypi.org/project/django-incident-response/)
package, a slack app (this needs configuring in your environment), a postgres
db used by the app to store incident data, the main response app and a cron
configured to hit an endpoint in Response every minute.

### Deployment 

#### Slack app
The slack app needs a number of scopes as listed
[here](https://github.com/monzo/response/blob/master/docs/slack_app_create.md#creating-your-slack-app).
Authorisation is handled by Slack through OAuth (see #Authentication) using
secret IDs and token which will need to be securely stored.
The app will need to be installed in the relevant slack workspace.

#### Server
A pre-requisite to the Slack app is a server running and accessible to the
internet.
In our case it will be deployed in a namespace in the nonprod EKS cluster
exposed through a k8s service type LoadBalancer and Cloudflare Loadbalancer.

#### Release 
The app seems to be updated once every month or two. Keeping up with the
updates will not be problematic.

### Authentication
Communication between application and slack is managed in Slack through [signed
secrets](https://api.slack.com/authentication/verifying-requests-from-slack).
Slack signs its requests using a secret key that is unique to the app; this
helps the app authenticate requests from Slack. A Slack OAuth Access token is
used to use the Slack API.

---

Django==2.2.13
psycopg2-binary==2.8.2
