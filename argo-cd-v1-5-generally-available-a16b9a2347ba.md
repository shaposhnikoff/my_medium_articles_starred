Unknown markup type 10 { type: [33m10[39m, start: [33m192[39m, end: [33m206[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m214[39m, end: [33m224[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m14[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m4[39m, end: [33m18[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m167[39m, end: [33m191[39m }
Unknown markup type 10 { type: [33m10[39m, start: [33m52[39m, end: [33m57[39m }

# Argo CD v1.5 Generally Available!

We are happy to announce Argo CD v1.5 release general availability.

### Better Performance

Argoproj team keeps working hard on improving Argo CD scalability and reliability! If you are running Argo CD instances with several hundred applications on it, you should see a huge performance boost and significantly less Kubernetes API server load. The performance improvement has been achieved by introducing read-write locks for the cluster state cache. The read-write locks enable concurrent access for read-only operations that eliminated most lock contention and improved reconciliation performance. The figure below is a reconciliation duration heat-map of the production Argo CD instance over time.

![](https://cdn-images-1.medium.com/max/2400/1*-t04ay5sj6BeelPM13F-RA.png)

The y-axis represents the average application reconciliation duration in seconds and the color represents the relative number of reconciliations that fall into a given duration bucket. As you can see, before the upgrade at 13:23, most of the users were seeing around 2-4 seconds delay. After the upgrade, most of the reconciliations are done in less than 0.25 seconds.

### Helm Support Improvements

![](https://cdn-images-1.medium.com/max/2000/1*W_hpvkY4ybSiv3YFc3YZbQ.png)

This release introduces Helm 3 charts support. Although most Helm charts are still Helm 2 compatible, it is important to be ready for Helm 3. Argo CD will use Helm 3 to render charts that use apiVersion: v2 in the Chart.yaml file. Helm 2 charts will continue to be rendered using the Helm 2 binary.

In addition to Helm 3 support, this release introduces two new Helm settings:

* The --set-file flag support. The flag allows files outside of the chart directory to be referenced and used via a chart.

* The --api-versions flag support. Argo CD now supplies information about supported Kubernetes versions to the Helm client. This enables support for charts that use the Capabilities.APIVersions object.

### Local Accounts

Argo CD supports SSO integration and has a built-in admin user that has full system access. As a result, the project never implemented its own user management. The new [local accounts](https://argoproj.github.io/argo-cd/operator-manual/user-management/#local-usersaccounts-v15) feature enables local account creation primarily to support access for automation and is not intended as an SSO replacement or a complete user management solution.

The main use-case of this feature is to generate auth tokens for Argo CD management automation. Using this feature, you can create an account with limited permissions and generate an access token that can be used to manage Argo CD configuration and to create projects and applications.

The local accounts can also be used to provide access to end-users for small teams where SSO is overkill. The local users don’t provide advanced features such as groups, login history etc. So if you need such features it is strongly recommended to use SSO.

### Windows CLI

Windows users deploy to Kubernetes too! Now you can use Argo CD CLI on Linux, Mac OS, and Windows. The Windows compatible binary is available in the release details page as well as on the Argo CD Help page.

### Redis HA Proxy mode

As part of this release, the bundled Redis was upgraded to version 4.3.4 with enabled HAProxy.
The HA proxy replaced the sentinel and provides more reliable Redis connection.
> After publishing 1.5.0 release we’ve discovered that default HAProxy settings might cause intermittent failures.
See [argo-cd#3358](https://github.com/argoproj/argo-cd/issues/3358)

## Where Can I Get This?

For more details and installation instructions, check the [v1.5 release notes](https://github.com/argoproj/argo-cd/releases/tag/v1.5.0).

## Thanks to the Community!

Thanks to our fantastic contributors for this release and our wonderful users for their feedback!
