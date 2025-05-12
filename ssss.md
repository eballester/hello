
```mermaid
graph TD
    %% Secció Global
    subgraph Global
        Internet((Usuari/Internet)) --> FD[Azure Front Door<br>padeltTransportistas];
        FD -- Protecció WAF --> FD_WAF(Directiva WAF<br>palprest001WAF);

        subgraph Zones DNS Privades
            PrivDNS_DB[privatelink.database.windows.net]
            PrivDNS_Cosmos[privatelink.documents.azure.com]
            PrivDNS_KV[privatelink.vaultcore.azure.net]
            PrivDNS_ACA[azurecontainerapps.io]
            PrivDNS_Gencat[gencat.cat]
            PrivDNS_Intranet[intranet.gencat.cat]
        end

        subgraph Vincles VNet
            Link_DB(pal-pre-wa-sqldb-001-zonelink)
            Link_Cosmos(pal-pre-wa-cosmosdb-001-zonelink)
            Link_ACA(palpre_link_001)
            Link_Gencat(palpre_link_vnet_001)
            Link_Intranet(palpre_link_intranet_001)
        end

        Link_DB --> PrivDNS_DB;
        Link_Cosmos --> PrivDNS_Cosmos;
        Link_ACA --> PrivDNS_ACA;
        Link_Gencat --> PrivDNS_Gencat;
        Link_Intranet --> PrivDNS_Intranet;

        AI_Global[AI Smart Detection<br>Grup d'Accions]
    end

    %% Secció Regional - West Europe
    subgraph West Europe

        subgraph Xarxa Virtual (Implicita)
            direction LR

            AG[Application Gateway<br>pal-pre-appgateway-001]
            VM[Màquina Virtual<br>pal-pre-vm]
            VM_NIC(pal-pre-nic)
            VM --> VM_NIC;

            subgraph Punts de Connexió Privats
                PE_SQL[(PE SQL)]
                PE_Cosmos[(PE Cosmos)]
                PE_Storage[(PE Storage)]
                PE_SFTP[(PE SFTP)]
                PE_KV[(PE Key Vault)]
                PE_CR[(PE CR)]
            end

            VM --> PE_SQL;
            VM --> PE_Cosmos;
            VM --> PE_Storage;
            VM --> PE_SFTP;
            VM --> PE_KV;

            ACAEnv[Entorn Container Apps<br>palprecappenv]
            ACAEnv --> PE_SQL;
            ACAEnv --> PE_Cosmos;
            ACAEnv --> PE_Storage;
            ACAEnv --> PE_SFTP;
            ACAEnv --> PE_KV;
            ACAEnv --> PE_CR;


        end %% End VNet

        subgraph Capa de Càlcul
            ACA_API[Aplicació Contenidora<br>palprecappapi001]
            ACA_Gestio[Aplicació Contenidora<br>palprecappgestio001]
            ACA_Transp[Aplicació Contenidora<br>palprecapptransportista001]
            ACA_API --> ACAEnv;
            ACA_Gestio --> ACAEnv;
            ACA_Transp --> ACAEnv;

            SWA_Admin[Aplicació Web Estàtica<br>palprest-sw-administracion-001]
            SWA_Transp[Aplicació Web Estàtica<br>palprest-sw-transportistas-001]
        end

        subgraph Capa de Dades
            SQLDB(Base de dades SQL<br>pal-pre-001)
            CosmosDB{{Azure Cosmos DB<br>pal-pre-001-cosmosdb}}
            StorageAccount[/Compte Emmagatzematge<br>palprest001/]
            StorageSFTP[/Compte Emmagatzematge SFTP<br>palprest001sftp01/]
            StorageTFState[/Compte Emmagatzematge<br>paltfstateprest/]
        end

        subgraph Capa de Seguretat
             KeyVault(((Almacén de Claves<br>pal-pre-keyvault-001)))
        end

         subgraph Contenidors
             ContainerRegistry(Container Registry<br>palprecr001)
         end

        subgraph Monitorització
            AppInsights[Application Insights<br>pal-pre-appi-001_insigt]
            LogAnalytics[Log Analytics Workspaces<br>pal-pre-appi-001wsp, palpre-law-001]
        end

        %% Connexions dins West Europe i a la VNet
        AG -- "Enruta trànsit" --> ACA_API;
        AG -- "Enruta trànsit" --> ACA_Gestio;
        AG -- "Enruta trànsit" --> ACA_Transp;


        PE_SQL --> SQLDB;
        PE_Cosmos --> CosmosDB;
        PE_Storage --> StorageAccount;
        PE_SFTP --> StorageSFTP;
        PE_KV --> KeyVault;
        PE_CR --> ContainerRegistry;

        ACA_API --> AppInsights;
        ACA_Gestio --> AppInsights;
        ACA_Transp --> AppInsights;
        SWA_Admin --> AppInsights;
        SWA_Transp --> AppInsights;
        VM --> AppInsights; % Si instrumentada

        AppInsights --> LogAnalytics;
        ACAEnv --> LogAnalytics; % Logs plataforma
        VM --> LogAnalytics; % Logs VM
        AG --> LogAnalytics; % Logs AG


        VNet -- "Resolving Private DNS" --> Link_DB;
        VNet -- "Resolving Private DNS" --> Link_Cosmos;
        VNet -- "Resolving Private DNS" --> Link_ACA;
        VNet -- "Resolving Private DNS" --> Link_Gencat;
        VNet -- "Resolving Private DNS" --> Link_Intranet;

    end %% End West Europe

    %% Connexions entre Global i Regional
    FD --> AG; % Trànsit dinàmic/API
    FD --> SWA_Admin; % Trànsit estàtic
    FD --> SWA_Transp; % Trànsit estàtic

    LogAnalytics --> AI_Global;

    %% Connexions per Private Endpoints (visualment des del PE dins la VNet al servei fora)
    PE_SQL -- "Accés Privat" --> SQLDB;
    PE_Cosmos -- "Accés Privat" --> CosmosDB;
    PE_Storage -- "Accés Privat" --> StorageAccount;
    PE_SFTP -- "Accés Privat" --> StorageSFTP;
    PE_KV -- "Accés Privat" --> KeyVault;
    PE_CR -- "Accés Privat" --> ContainerRegistry;
```
