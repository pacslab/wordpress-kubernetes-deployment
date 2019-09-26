# To Deploy Custom Load Tester

In case you want the load tester to have a different behaviour than the default, do the following steps:

- Update the `locustfile.py` with your custom configuration.

- Update `build.sh` with your docker hub username instead of `nimamahmoudi` on both lines.

- Run the following:

```sh
source build.sh
```

This will build and push the custom image to your account, then you can use the generated image in your kubernetes cluster.