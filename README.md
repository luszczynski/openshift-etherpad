# Etherpad on Openshift

This is a simple repo describing how you can deploy etherpad-lite for your workshops.

![](imgs/2021-08-10-10-46-42.png)

## Instructions

### Installation

Tested on OpenShift 4.12 (make sure you `oc` command line is on 4.12 version. You can check that by running `oc version`)

```bash
ETHERPAD_PROJECT=etherpad
ETHERPAD_VERSION=1.8.18
ETHERPAD_APP_NAME=etherpad

# Creating project
oc new-project ${ETHERPAD_PROJECT} --display-name "Etherpad"

oc new-app \
    --template=postgresql-persistent \
    --param DATABASE_SERVICE_NAME=postgresql \
    --param POSTGRESQL_USER=ether \
    --param POSTGRESQL_PASSWORD=ether \
    --param POSTGRESQL_DATABASE=ether \
    --param POSTGRESQL_VERSION=10 \
    --param VOLUME_CAPACITY=2Gi \
    --labels=app=database \
    -n ${ETHERPAD_PROJECT}

# Wait for the database to be ready
sleep 45

# Creating etherpad for Postgresql
oc new-app \
    --name=${ETHERPAD_APP_NAME} \
    docker.io/etherpad/etherpad:${ETHERPAD_VERSION} \
    DB_TYPE=postgres \
    DB_HOST=postgresql \
    DB_PORT=5432 \
    DB_USER=ether \
    DB_PASS=ether \
    DB_NAME=ether \
    ADMIN_PASSWORD=supersecret \
    --labels=app=etherpad \
    -n ${ETHERPAD_PROJECT}

# Creating route for etherpad
oc expose svc ${ETHERPAD_APP_NAME} -n ${ETHERPAD_PROJECT}

# Grouping postgresql and etherpad in the same application
oc label deploy ${ETHERPAD_APP_NAME} app.kubernetes.io/part-of=${ETHERPAD_APP_NAME} -n ${ETHERPAD_PROJECT}
oc label dc postgresql app.kubernetes.io/part-of=${ETHERPAD_APP_NAME} -n ${ETHERPAD_PROJECT}
oc annotate deploy ${ETHERPAD_APP_NAME} app.openshift.io/connects-to=postgresql-persistent -n ${ETHERPAD_PROJECT}

# On MacOs
open http://$(oc get route ${ETHERPAD_APP_NAME} -o jsonpath='{.spec.host}')
```

You should see now the following screen

![](imgs/2020-05-27-12-41-04.png)

To open Etherpad, click on the arrow for the Etherpad pod:

![](imgs/2021-08-10-10-48-46.png)

### Creating list of Users

Use the following script to generate a list of users.

```bash
# Change this according to the number of users
NUMBER_OF_USERS=20

for userNumber in $(seq 1 $NUMBER_OF_USERS);do
  echo "user${userNumber}="
done
```

You should see the following output. Use it in your etherpad.

```bash
user1=
user2=
user3=
user4=
user5=
user6=
user7=
user8=
user9=
user10=
user11=
user12=
user13=
user14=
user15=
user16=
user17=
user18=
user19=
user20=
```
