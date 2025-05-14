# Jenkins GitOps Kubernetes Repository

Welcome to the Jenkins GitOps Kubernetes repository! This repository contains the Kubernetes deployment and service manifests necessary for deploying applications using Jenkins in a GitOps workflow.

## Overview

In this repository, you'll find deployment and service manifests that define how applications are deployed and exposed within a Kubernetes cluster. These manifests are managed and updated automatically through Jenkins pipelines, ensuring continuous integration and delivery.

## Getting Started

To get started with deploying applications using Jenkins and Kubernetes, follow these steps:

1. **Clone the Repository:** Clone this repository to your local machine.

2. **Configure Jenkins Pipeline:** Set up Jenkins to trigger pipelines based on changes to this repository. Refer to our [medium article](https://medium.com/@sameeradissanayaka/building-a-robust-ci-cd-pipeline-with-jenkins-docker-kubernetes-and-argocd-bdcc15a31a2f) for detailed instructions on setting up a robust CI/CD pipeline with Jenkins, Docker, Kubernetes, and ArgoCD.

3. **Update Manifests:** Customize the deployment and service manifests according to your application requirements. Ensure that the paths and image references are correctly configured.

4. **Commit Changes:** Once you've made the necessary changes to the manifests, commit them to this repository.

5. **Monitor Deployment:** Jenkins will automatically detect changes in this repository and trigger pipeline jobs to deploy your applications to the Kubernetes cluster. Monitor the pipeline execution and deployment status in Jenkins.

## Additional Resources

For more information on setting up a complete CI/CD pipeline with Jenkins, Docker, Kubernetes, and ArgoCD, check out our comprehensive guide on Medium:
[Building a Robust CI/CD Pipeline with Jenkins, Docker, Kubernetes, and ArgoCD](https://medium.com/@sameeradissanayaka/building-a-robust-ci-cd-pipeline-with-jenkins-docker-kubernetes-and-argocd-bdcc15a31a2f)

# Own documentation. 
First than all we need to set up the driver that minikube must use, in this case we use KVM, is important to say that to change driver we must reboot when we apply the  `minikube start --driver=kvm2`.

![image](https://github.com/user-attachments/assets/e4c9a4e0-2b12-4bb0-9bb3-923e1c17c928)

---
Then, for argos installation we use a predefine manifest for it so we use the command: `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`, after that we can check the pods created by the manifest wit the command `kubectl get pods -n argocd`:

![image](https://github.com/user-attachments/assets/dc756909-4958-4e72-9d14-4930ee5135e2)

To acces argos we need to do a port forwarding with the commmand: `kubectl port-forward svc/argocd-server -n argocd 8081:443`,  must differ from 8080 well this is the default port for jenkins when we install it from  service systemd.

![image](https://github.com/user-attachments/assets/d82d7714-e30f-4a17-be98-fdf10444815b)

Now we need to get the secret to acces argos dashboard,  for it we use the command: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath={.data.password} | base64 -d`,  the we use it as a password and we can check argos dashboards:

![image](https://github.com/user-attachments/assets/7f274b83-4256-4bcc-ae05-3ca276804b06)

---
# Jenkins as a systemd Service

In this setup, Jenkins runs under systemd like any other Linux service. After installing Jenkins, the initial administrator password is stored on the filesystem:

##Install Jenkins (on Ubuntu/Debian):
```
sudo apt update
sudo apt install -y openjdk-11-jdk jenkins
sudo systemctl enable --now jenkins
```
Find the Initial Admin Password

`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

Copy this long alphanumeric string to unlock the Jenkins setup wizard in your browser.

Now, we need to refer the pipeline of each repository with SCM :

![image](https://github.com/user-attachments/assets/f9e74ec2-2f36-49c7-8ccc-f3d3e1ad5806)

---
# Argos App creation process: 

Refer to the repository that contains the flask app: 

![image](https://github.com/user-attachments/assets/929cd8c8-24eb-4f23-9647-f28ae522bf79)

Checking app status bases on the repo reference:

![image](https://github.com/user-attachments/assets/45abd40c-339e-423a-9b4b-1bde5208f8b8)

Now, in orde to acces the app we need to expose a loadbalancer with `minikube service lb-service`:

![image](https://github.com/user-attachments/assets/fb65b0ba-ab13-4d3a-96c1-3a5c0c7b4922)

Also, I want to expose my local Jenkins with ngrook, in order to do that we need to expose my `localhost:8080`: 

![image](https://github.com/user-attachments/assets/d3dddd99-98f6-4e1b-818f-0a475a38a57b)

And, then we create a webhook on the repository based on the url given by ngrook,  is important to add `/` at the end:

![image](https://github.com/user-attachments/assets/4f7ef584-24f6-44ed-94ec-0494dd097597)

Now based on a push to main, the jenkinsfile that is in charge of commit the new version of the manifest will apply:

![image](https://github.com/user-attachments/assets/2d28c804-e370-4ef5-9d0c-bb5e4061610a)

![image](https://github.com/user-attachments/assets/915312fe-a207-47c7-9dc1-4a9efb8a2b0c)
