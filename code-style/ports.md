# Ports

You need to book ports before adding services in this docs table.

The key take away is to provide a convention to make a conscious decision about ports for a new service and enable all product services parallel run on a developer's computer.

When you add a new service you need to add a new row at the end of this table as a copy of the previous one and increment each port by 1. For instance, the last service ports were: 4501, 5501, 6501, 7501, 8501, 9501 => the new ones will be 4502, 5502, 6502, 7502, 8502, 9502.

This approach should be scalable to make other products using this infra and approaches. In case of a new product we might use not 500 but 600. So the first service ports will be: 4601, 5601, 6601, 7601, 8601, 9601.

## Local

### UI-services

| Service Name                  | Dev Container/Codespaces        | IDE        | Docker Compose        |
| :-----------------------------| :-----------------------------: | :--------: | :-------------------: |
| inner-circle-layout-ui        |               4500              |    5500    |          6500         |
| inner-circle-items-ui	        |               3501*             |     -      |           -           |
| inner-circle-mentoring-ui     |               3502*             |     -      |           -           |
| auth-ui	                    |               3503*             |     -      |           -           |
| inner-circle-documents-ui	    |               3504*             |     -      |           -           |
| inner-circle-books-ui         |               3505              |     -      |           -           |
| inner-circle-ui	            |               3506*             |     -      |           -           |
| inner-circle-time-ui	        |               3507*             |     -      |           -           |
| inner-circle-invoices-ui	    |               3508*             |     -      |           -           |
| inner-circle-compensations-ui	|               3509*             |     -      |           -           |
| accounts-ui	                |               3510*             |     -      |           -           |

### API-services

| Service Name                   | Api in Dev Container/Codespaces | Api in IDE | Api in Docker Compose |  Db in Docker Compose | MockServer in Docker Compose | PgAdmin in Docker Compose |
| :----------------------------  | :-----------------------------: | :--------: | :-------------------: | :-------------------: | :-------------------------: | :-------------------------: |
| inner-circle-items-api         |               4501              |    5501    |          6501         |          7501         |             8501            |             9501            |
| inner-circle-mentoring-api     |               4502*             |    5502*   |          6502*        |          7502*        |             8502*           |             9502*           |
| auth-api                       |               4503*             |    5503    |          6503         |          7503         |             8503            |             9503            |
| inner-circle-documents-api     |               4504*             |    5504*   |          6504         |          7504         |             8504            |             9504*           |
| inner-circle-books-api         |               4505              |    5505    |          6505         |          7505         |             8505            |             9505            |
| inner-circle-employees-api     |               4506*             |    5506*   |          6506         |          7506         |             8506            |             9506*           |
| inner-circle-time-api          |               4507              |    5507    |          6507         |          7507         |             8507            |             9507            |
| inner-circle-invoices-api      |               4508              |    5508    |          6508         |          7508*        |             8508            |             9508*           |
| inner-circle-compensations-api |               4509*             |    5509*   |          6509*        |          7509*        |             8509*           |             9509*           | 
| accounts-api	                 |               4510*             |    5510    |          6510         |          7510         |             8510            |             9510            |

> *- Changes to ports have not yet been made to the service code
