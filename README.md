# mta_python_ml


Video: [Introducing the Python Client API for SAP HANA](https://video.sap.com/media/t/1_0bw54r9a/)


[Neural Networks and Deep Learning](http://neuralnetworksanddeeplearning.com/)


[Code samples for "Neural Networks and Deep Learning"](https://github.com/mnielsen/neural-networks-and-deep-learning)


Better Handwriting Recognition Example:
Git: [DeepLearningPython35](https://github.com/MichalDanielDobrzanski/DeepLearningPython35)


```
pip download -d vendor -r requirements.txt --find-links ../../sap_dependencies --find-links ../../hana_ml-1.0.3.tar.gz hana_ml
../../mta_schemaless/tools/minion/minion -unead -t xsa
```

Start/Stop WebIDE

```
xs t -o HANAExpress -t SAP
xs start devx-ui5 ; xs start tools ; xs start webide
xs stop webide ; xs stop tools ; xs stop devx-ui5
```

Currently using these steps.

```
xs t -o HANAExpress -s ml
```

Create service instances

```
xs create-service hana hdi-shared python-ml-hdi
xs create-service xsuaa default python-ml-uaa
```

Manually build the DB module

```
git pull
xs push python-ml.db -k 1024M -m 256M -p db --no-start --no-route
xs bind-service python-ml.db python-ml-hdi
xs restart python-ml.db --wait-indefinitely ; sleep 15 ; xs stop python-ml.db
--or--
cd db ; npm install ; cd ..

git pull
xs push python-ml.db -k 1024M -m 256M -p db --no-start --no-route
xs restart python-ml.db --wait-indefinitely ; sleep 15 ; xs stop python-ml.db
```

Manually build the python module

```
git pull
xs push python-ml.python -k 1024M -m 256M -n python -p python --no-start
xs bind-service python-ml.python python-ml-hdi
xs bind-service python-ml.python python-ml-uaa
xs start python-ml.python

git pull ; xs push python-ml.python -k 1024M -m 256M -n python -p python
```

Manually build the xsjs module

```
git pull
xs push python-ml.xsjs -k 1024M -m 256M -n xsjs -p xsjs --no-start
xs bind-service python-ml.xsjs python-ml-hdi
xs bind-service python-ml.xsjs python-ml-uaa
xs start python-ml.xsjs

git pull
xs push python-ml.xsjs -k 1024M -m 256M -n xsjs -p xsjs
```

Manually build the web module

```
cd web ; npm install ; cd ..
git pull
xs push python-ml.web -k 1024M -m 256M -n web -p web --no-start
xs bind-service python-ml.web python-ml-uaa
xs set-env  python-ml.web destinations '[{"forwardAuthToken":true, "name":"python_be", "url":"https://hxehost:51051"}]'
xs start python-ml.web

git pull ; xs push python-ml.web -k 1024M -m 256M -p web ; xs restart python-ml.web --wait-indefinitely
```

```
echo "Changing to running app directory." ; pyguid=$(xs app python-ml.python --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; echo $pyguid ; cd /hana/shared/HXE/xs/app_working/hxehost/executionroot/$pyguid/app

echo "Saving server.py to project." ; cp server.py ~/../HDB90/hxe_python_ml/mta_python_ml/python/
echo "Loading server.py from project." ; cp ~/../HDB90/hxe_python_ml/mta_python_ml/python/server.py .

Watch python
pyguid=$(xs app python-ml.python --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; pydir=$(echo "/hana/shared/HXE/xs/app_working/hxehost/executionroot/$pyguid/app") ; ./xsa_exec_watch.sh python $pydir


echo "Changing to running app directory." ; pyguid=$(xs app python-ml.web --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; echo $pyguid ; cd /hana/shared/HXE/xs/app_working/hxehost/executionroot/$pyguid/app

echo "Saving index.html to project." ; cp index.html ~/../HDB90/hxe_python_ml/mta_python_ml/web/resources/
echo "Loading index.html from project." ; cp ~/../HDB90/hxe_python_ml/mta_python_ml/web/resources/index.html .

Watch web
webguid=$(xs app python-ml.web --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; webdir=$(echo "/hana/shared/HXE/xs/app_working/hxehost/executionroot/$webguid/app/resources") ; ./xsa_exec_watch.sh web/resources $webdir

```

