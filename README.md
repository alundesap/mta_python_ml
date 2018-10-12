# mta_python_ml

```
pip download -d vendor -r requirements.txt --find-links ../../sap_dependencies --find-links ../../hana_ml-1.0.3.tar.gz hana_ml
../../mta_schemaless/tools/minion/minion -unead -t xsa
```

Currently using these steps.

```
xs create-service hana hdi-shared python-ml-hdi
xs create-service xsuaa default python-ml-uaa

xs push python-ml.db -k 1024M -m 256M -p db --no-start --no-route
xs bind-service python-ml.db python-ml-hdi
xs restart python-ml.db --wait-indefinitely ; sleep 10 ; xs stop python-ml.db

xs push python-ml.python -k 1024M -m 256M -n python -p python --no-start
xs bind-service python-ml.python python-ml-hdi
xs bind-service python-ml.python python-ml-uaa
xs start python-ml.python

```

