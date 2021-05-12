# Customizing and Using the Internal OCP Registry

## Let's access the internal OpenShift 4.x registry
---
##### Note: The internal registry is primarily used to distribute images throughout a single cluster. Accessing this internal registry for CI could be handy as it integrates with your existing users and groups, however, care should be taken as normal garage collection and clean up of this registry is not automated. It's a great way to start out your test/dev/uat clusters, but do consider an enterprise registry for production - like Nexus, Docker Trusted Registry, Artifactory or Cloud providers like Docker Hub, Red Hat, Azure, Google, AWS and others. Having an external registry allows you to scale across clusters, integrate with more granular authentication practices, perform CV scanning and image analysis, garbage collection and basic hygiene of shared image layers.

---
1. Expose the route to the internal registry. \
`oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge`

2. TLS - Accessing the registry should be secure with your domain certificates and key, these could be self-signed, just specific for the route you'll add in a later step. If you choose to use the default route, the *.apps.<cluster>.<domain.com> certificate is fine. \
`oc create secret tls public-route-tls -n openshift-image-registry --cert=</path/to/tls.crt> --key=</path/to/tls.key>`

3. Set your custom TLS route/cert. \
`oc edit configs.imageregistry.operator.openshift.io/cluster`
    ```
    spec:
    routes:
      - name: public-routes
        hostname: registry.domain.com
        secretName: public-route-tls
    ```
4. Verify the route was added to the registry. \
`oc -n openshift-image-registry get route`

    ```
    example:
    NAME            HOST/PORT                                                             PATH   SERVICES         PORT    TERMINATION   WILDCARD
    default-route   default-route-openshift-image-registry.apps.openshift.redcloud.land          image-registry   <all>   reencrypt     None
    public-routes   registry.redcloud.land                                                       image-registry   <all>   reencrypt     None
    ```

5. Let's now test access to the registry.
   Remember, it's by 'namespace'/'project' so we'll first create a project, assign our registry-editor role to that project and then push an image into the namespace.

   i. `oc new-project reg-test` \
   ii. `oc -n reg-test policy add-role-to-user registry-editor shaker` \
   iii. `docker pull alpine` \
   iv. `docker tag alpine:latest registry.redcloud.land/reg-test/nginx:latest` \
   v. `docker login -u shaker registry.redcloud.land` # use API token \
   vi. `docker push registry.redcloud.land/reg-test/nginx:latest` \
   vii. `oc -n reg-test get is` 

More info: 
OCP 4.7 Registry Options: https://docs.openshift.com/container-platform/4.7/registry/registry-options.html

