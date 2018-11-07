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
The following commands are super useful in doing active development on XSA.

First, understand that when you push or deploy your module, it's getting stuffed into the blobstore and then copied out of the blobstore into a "container" using process separation.  To find and change into the folder your module is running in, use this command.

```
echo "Changing to running app directory." ; pyguid=$(xs app python-ml.python --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; echo $pyguid ; cd /hana/shared/HXE/xs/app_working/hxehost/executionroot/$pyguid/app
```
Now you can edit your files and save them here to test different but you have to be careful since as noted above, these files are copied out of the blobstore.  If you end up crashing the process for creating a syntax error, the current instance of your module will be abandoned and the system will create a new one.  Just run the command above again and you'll get into the latest directory of your running module.  Note also that this may not work as expected if you request multiple instances of your module.  If your module crashes, you'll loose all the changes you've made since the last push/deploy operation.  To get around this, you can copy your files back to the place where you built them like so.

```
echo "Saving server.py to project." ; cp server.py ~/../HDB90/hxe_python_ml/mta_python_ml/python/
echo "Loading server.py from project." ; cp ~/../HDB90/hxe_python_ml/mta_python_ml/python/server.py .
```
However, this is a bit painful since you have to remember to do this BEFORE you make a mistake.

A better approach is to work from your project dirctory and then copy your changed files into the active directory.  If your python module is in debugging mode, it will pick up that the file has changed and will automatically reload itself.

Even better is to set up a script to watch your project files and copy them into the active directory whenever you save them.

This is exactly what the following script does.  It also frees you from having to worry about forgetting to copy the changed files.

Perform this from the main project folder.

Watch python files.
```
cd /usr/sap/HXE/HDB90/hxe_python_ml/mta_python_ml
pyguid=$(xs app python-ml.python --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; pydir=$(echo "/hana/shared/HXE/xs/app_working/hxehost/executionroot/$pyguid/app") ; ./xsa_exec_watch.sh python $pydir
```
Now leave that console window running and go edit your files in another console window.
```
vi python/server.py
```
Same thing works for your web module.

```
echo "Changing to running app directory." ; pyguid=$(xs app python-ml.web --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; echo $pyguid ; cd /hana/shared/HXE/xs/app_working/hxehost/executionroot/$pyguid/app

echo "Saving index.html to project." ; cp index.html ~/../HDB90/hxe_python_ml/mta_python_ml/web/resources/
echo "Loading index.html from project." ; cp ~/../HDB90/hxe_python_ml/mta_python_ml/web/resources/index.html .

Watch web
webguid=$(xs app python-ml.web --guids | grep RUNNING | tr -s " " | cut -d " " -f 12) ; webdir=$(echo "/hana/shared/HXE/xs/app_working/hxehost/executionroot/$webguid/app/resources") ; ./xsa_exec_watch.sh web/resources $webdir

```

