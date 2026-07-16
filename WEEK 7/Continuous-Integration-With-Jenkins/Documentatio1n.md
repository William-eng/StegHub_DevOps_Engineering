# Configure Ansible configuration repo For Jenkins Deployment 
This is project is continuation from my ansible dynamic assignment projects. I would be using a copy of the ansible-config-mgt repo which I have renamed ansible-configuration

## STEP ONE: Spinning up our and Connecting to Our Jenkins Server
We'll Start by spinning up our stopped AWS EC2 instance with a Ubuntu OS with Jenkins set up.
- ![SpinupJenkins](https://github.com/user-attachments/assets/8ca88b8e-b9cc-4563-884c-f3867d3a49ec)
## STEP TWO : Installing Blue-Ocean Plugin
To make managing our Jenkins pipelines easier and more interactive UI, we will install the Blue Ocean plugin. This plugin offers a user-friendly and visually appealing interface, 
helping us quickly understand the status of your continuous delivery pipelines.
To do this , we'll follow the steps below :

- Go to manage jenkins > manage plugins > available
- Search for BLUE OCEAN PLUGIN and install
- ![blueOcean](https://github.com/user-attachments/assets/9be29713-6c92-4699-95ff-3f257621d00c)































































































