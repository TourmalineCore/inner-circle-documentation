# Migration of Kubernetes from One Cloud to Another

## Objectives

- Pay less for low-demand enterprise informational system hosting.
- Be able to reproduce prod from manifests instead of manual patching of the cluster.
- Prepare for Migration from Cloud to On-Premises using corporate servers in Intranet.

## Where We are

- Current cloud provides k8s cluster as a service
- Current cloud provides PostgreSQL RDMS as a service
- We cannot reproduce production easily using local-env or prod-env (doesn't exist).
- Each service deploys itself to production.
- Shared services like e.g. ingress or cert-manager were deployed to Prod manually.
- We lost the possibility to stop the cluster for week-end and at night (was implemented previously using open stack calls to the current cloud internal APIs, half-hack).

## Where We Want to Be

- All services and their data are hosted On-Premises using corporate servers in Intranet.

## Roadmap

1. Create inner-circle-cloud with 1 VM (2 CPU 5% (the lowest possible) and static IP-address, 4GB RAM) in it using home-cloud as an example. Setup resources in the target cloud using it (local run, no CI/CD)
1. Create inner-circle-env with prod environment in it that contains:
ingress,
cert-manager,
metric-server,
1. Deploy inner-circle-env with its prod environment to the new cluster (local run, no CI/CD)
1. Add needed DNS records for the new Inner Circle Prod domain (new one)
**!!! At this point it should show requests to ingress in its logs and LetsEncrypt https certificates are issued**
1. Spin up Intranet GitHub Actions self-hosted runners for IC deployment and prod e2e tests only. They will have cluster's KUBECONFIG stored in the filesystem.
1. Add a prod deployment workflow to inner-circle-env that will use the new self-hosted runners to deploy it to the new prod on a master branch push.
**!!! At this point we have tested the deployment from the self-hosted runners that contain KUBECONFIG in their filesystem.**
1. IMPORTANT: Override INNER_CIRCLE_PROD_NAMESPACE with new namespace in every service repo.
1. Migrate inner-circle-employees-api workflows and all its Secrets to inner-circle-items-api.
1. Migrate inner-circle-employees-ui workflows and all its Secrets to inner-circle-time-ui.
1. Freeze any changes to inner-circle-employees-api and inner-circle-employees-ui.
1. Allow to connect to RDMS from Internet from the new cloud VM static IP (not only internally inside the first cloud local network)
1. Deploy inner-circle-employees-api to the new cloud while keeping it in the old as well.
It should be deployed only from the new self-hosted runners. Thus, we aren't storing KUBECONFIG from the new cluster in GitHub.
e2e tests should be run from the new self-hosted runners. Thus, we are going towards not storing secrets in GitHub at all. We will have access to logs of failed tests in Intranet since we have to hide it in the public workflow logs.
It should connect to its DB using RDMS public IP-address (currently it is using local network address of Host).
If it depends on any other IC service it should call it using IC public domain instead of service name (maybe there no such deps).
1. Deploy inner-circle-employees-ui to the new cloud while keeping it in the old as well. It is assumed that layout-ui temporarily has * for CORS and will allow to call it.
**!!! At this point it should be possible to open employees-ui on the new domain after manually putting accessToken and refreshToken from the old domain prod to LocalStorage in a browser. employees-ui should show the same data and allow to make changes as on the old prod.**
1. Deploy inner-circle-items-api to the new cloud:
It should re-use the steps for inner-circle-employees-api.
It should use internal address of inner-circle-employees-api service (not public) since they are now in the same cluster.
1. Remove inner-circle-items-api from the old cloud deployment. It is safe since there are no clients of it except for prod e2e tests.
1. Freeze any changes to auth-ui, auth-api, accounts-ui, accounts-api.
1. Deploy auth-ui, auth-api, accounts-ui, accounts-api while keeping it in the old as well.
1. Switch inner-circle-employees-ui, inner-circle-employees-api, inner-circle-items-api to internal service address where we had to temporarily change it to public IC domain addresses.
**!!! At this point we should get a tiny version of IC in the new cloud targeting the same DB as the old cloud. Thus, both versions of IC for the same services should work equivalently and should not lead to data inconsistency.**
1. Migrate the rest of services to the new cloud freezing the changes in them before that.
1. Remove the old cloud.

ToDo DB Migration.

 
