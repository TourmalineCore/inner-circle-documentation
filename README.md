# inner-circle-documentation

## Links to all project repositories

| Repository                                                                                         | Comment    |
|----------------------------------------------------------------------------------------------------|------------|
| [auth-ui](https://github.com/TourmalineCore/auth-ui)                                               |            |
| [auth-api](https://github.com/TourmalineCore/auth-api)                                             |            |
| [accounts-ui](https://github.com/TourmalineCore/accounts-ui)                                       |            |
| [accounts-api](https://github.com/TourmalineCore/accounts-api)                                     |            |
| [inner-circle-documents-ui](https://github.com/TourmalineCore/inner-circle-documents-ui)           |            |
| [inner-circle-documents-api](https://github.com/TourmalineCore/inner-circle-documents-api)         |            |
| [inner-circle-ui](https://github.com/TourmalineCore/inner-circle-ui)                               |            |
| [inner-circle-employees-api](https://github.com/TourmalineCore/inner-circle-employees-api)         |            |
| [inner-circle-compensations-ui](https://github.com/TourmalineCore/inner-circle-compensations-ui)   |            |
| [inner-circle-compensations-api](https://github.com/TourmalineCore/inner-circle-compensations-api) |            |
| [inner-circle-email-sender](https://github.com/TourmalineCore/inner-circle-email-sender)           |            |
| [inner-circle-books-ui](https://github.com/TourmalineCore/inner-circle-books-ui)                   |            |
| [inner-circle-books-api](https://github.com/TourmalineCore/inner-circle-books-api)                 |            |
| [inner-circle-items-api](https://github.com/TourmalineCore/inner-circle-items-api)                 |            |
| [inner-circle-layout-ui](https://github.com/TourmalineCore/inner-circle-layout-ui)                 |            |
| [inner-circle-local-env](https://github.com/TourmalineCore/inner-circle-local-env)                 |            |
| [inner-circle-reverse-proxy](https://github.com/TourmalineCore/inner-circle-reverse-proxy)         |            |

## Another useful links
- [Project board](https://github.com/orgs/TourmalineCore/projects/5)
- [USM board](https://board.mail.ru/?utm_source=portal&utm_medium=portal_navigation&utm_campaign=board.vk.com_cross_navimt_link_id%3Dykpiid2&uid=94acfbe7-8699-47d4-92df-fb705322e541)
- [Retro board](https://app.holst.so/board/047a6d87-4a7c-4179-840f-8c54f6fd21d2)
- [Figma designs](https://www.figma.com/design/O4Kbm638zRYHhVqN9FBm5D/Inner-Circle?t=kPjBwo15UyXLnic5-0)
- [Design vs Propotype discussion board] (https://app.holst.so/board/1629a6a2-f40c-488d-b038-7286e65fa4f4)

## C4 model

At the time the architectural diagrams were created, [Mermaid](https://mermaid.js.org/) did not yet fully and beautifully support C4 drawing. 
Therefore, [PlantUML](https://plantuml.com/en/stdlib) is used to draw C4 diagrams in this repository. Since C4 is not natively rendered with PlantUML in GitHub at the time the diagrams were published, the option of rendering the diagram as a photo is used, with the original `.puml` file with the diagrams saved in the `c4-architecture-diagrams` folder.


### Containers Diagram 

![Inner Circle Containers Diagram](https://www.plantuml.com/plantuml/png/dLPHRo8t47xdLxXvQP08dDG-LFTq2Q1hggHDGz2fJyZi3c35Qs_jsIDHrVzUUnOcwm8-BZmWpsZc-yRpsUFzv54wR2hT-q6PGZSbY0rYxJ-EHavkXsk5csRPU725DGW6XuMjHhMM9kPSUIorMex6FryzHDVy_cvKIO_WWcin3XbI6N33EkBt_BCojNIo4bidwo-gn8tuwSE7tjtrNt_CdnPVRfz-NTnDFqz_drpsUxqRsvXIkjVBMnCWnieL6lQ-WRWWhsZT5_r_UcBtuxJh1ZUT-6XUz0K5yeyhfEZvxIvEp6ylml8HP86ZtsPn_I3NuCNAEk5WhJmwYSgQWP0X70ogWcmXJIbgwVtMkZ94Ui3XHJz3UVt0d64PYOO6KFmILy51X4Seq3w34Mi8CArpgm92L4dKu2hbVN3wazIJ9P5FRlRSanlQe_MuTZ3xyovQCxGE_F_PbGabLRJ-ABHTApDus_16-ir001PTvdzxHxGfYh14tz5dxpgj4MUHJdiZgrKbZGmbheNSHG-4infysqGmqY1rASz78Z7XQQ7UpqQQC1_fmdGvRL8UEWNT0p3OitPb-vOsqwhMzXMWAmxxCkgGAcU9KJkxKXgeBLfv57ObOCUGaOEBQFLWCo_8knb94YOXJTZAYCAuF_hqiXXPmK0KrjNMIQHQebRU6gb3abJwhAipiKLJGNfts9UHXsD89xQorwVag2RiIPQXbLJsLwbLcHoc31-fnOGPjPWjZy_QC2HfmzOHRKWvV1K1KowbXs_rU6oh6eod2hfzJloPNYTnrE_4SzR8eaOx-Bi1Zz4ikWYyWImrjJxbphJ_dSIiHqfDcYIUp9r9ECuOLibGa4hFm9JW0XnPeZNJNKQVRw-P5sZVRh-mfdCfAQ0bPK8cMqnVyF8-1DpObm0PLqFNY9Yoj2-H9GI0CpAMuBwg0g8lW17C8sZ-ok2dFu0y0uaY2gcfD3eQqzaJ5K5fY_ab9wHlqcV0xhmd7UahW5w2mZe71Oh2GUaFOTz0-MFW4MGfLivMT0YsJY48jCTIqhvJT_axuCg1bWZbhjr90pfF-7vzVYCuFMd5UPKdgUbJ4fBfj5Fx2WTltrabQ233heBDX-PaHIphEAB0YFAXbKIkCFwtyKmpsbDqTilWQPbivJPy16NUoJsdsPoPlXbid8Je7oQUEYiyx0_FIegHLyop93JRa-8FXd3Y-8v3TvYNU79gv43EtH5396taCPHCDffqzqyF8UrJS3UL_Wy0)
