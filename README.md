# HUE Plugin

## How to setup
1. Startup LiDOP
2. Extend LiDOP with Selenium and Sonarqube:
    - Trigger the Build **Environment/Create_Service** with the following parameters:
        - **service**: Selenium
    - Trigger the Build **Environment/Create_Service** with the following parameters:
        - **service**: Sonarqube
3. Trigger the Build **LiDOP/Load Plugin** with the following properties:
    - **ProjectName:** HUE
    - **PluginUrl:** https://github.com/LivingDevOps/LiDOP.HUE.git 
    - **Branch:** */master
    - **Credential:** use the default (lidop)
    - **CopyRepository:** checked 
4. Wait until all 3 pipelines are done and success
5. Trigger the Build **HUE/Create_Environment** with the following properties:
    - **NodeRedLocation:** local or remote to define where LiDOP is installed
    - **HueBridgeIp:** the IP address of the HUE bridge
    - **HueDimmerId:** the id of the dimmer
    - **HueLightId:** the id of the HUE lamp 
