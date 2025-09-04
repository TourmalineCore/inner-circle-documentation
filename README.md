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
| [inner-circle-mentoring-api](https://github.com/TourmalineCore/inner-circle-mentoring-api)         |            |
| [inner-circle-layout-ui](https://github.com/TourmalineCore/inner-circle-layout-ui)                 |            |
| [inner-circle-local-env](https://github.com/TourmalineCore/inner-circle-local-env)                 |            |
| [inner-circle-reverse-proxy](https://github.com/TourmalineCore/inner-circle-reverse-proxy)         |            |

## Another useful links
- [Project board](https://github.com/orgs/TourmalineCore/projects/5)
- [USM board](https://board.mail.ru/?utm_source=portal&utm_medium=portal_navigation&utm_campaign=board.vk.com_cross_navimt_link_id%3Dykpiid2&uid=94acfbe7-8699-47d4-92df-fb705322e541)
- [Retro board](https://app.holst.so/board/047a6d87-4a7c-4179-840f-8c54f6fd21d2)
- [Figma designs](https://www.figma.com/design/O4Kbm638zRYHhVqN9FBm5D/Inner-Circle?t=kPjBwo15UyXLnic5-0)
- [Design vs Propotype discussion board](https://app.holst.so/board/1629a6a2-f40c-488d-b038-7286e65fa4f4)

## C4 model

At the time the architectural diagrams were created, [Mermaid](https://mermaid.js.org/) did not yet fully and beautifully support C4 drawing. 
Therefore, [PlantUML](https://plantuml.com/en/stdlib) is used to draw C4 diagrams in this repository. Since C4 is not natively rendered with PlantUML in GitHub at the time the diagrams were published, the option of rendering the diagram as a photo is used, with the original `.puml` file with the diagrams saved in the `c4-architecture-diagrams` folder.


### Containers Diagram 

![Inner Circle Containers Diagram](https://www.plantuml.com/plantuml/png/dLPTRoet47tdLxYyD4W4hjG-jFVf9e1kgP8s3K5NVK9cTm0hk_RMdWs9glxtshCmMHVYR8EYo3Fod3C-nn_xnQ6qM9T5xpkbiwBEGUmGA_TvFBPoFzegtDNhse7DZ4RGECfCEQuAgKFDfSEyKElnvCVBkv1QtjwCI-aGh4-j9hv2AWrsL0NyddjKkf9Mbc9lPVLDvRWJFtpwrBkv-lFtvSFgs_pXbzNDxElij-d5eDUxDhNEfNsvc6iFCL4sA-2O6ue-PKMJxW_-xed3ZykkQjorveUmFn0qaN_S8SrUpW_pn7BUzyDxa1cEVrs4yPtSWXCROuM5hN9e9IgZXqBwwQ1HPHGBgNDHIUVsnkP-fWCSzGS9ogixvlJ3G3JIWEAdCF898Zn3WUOHjDY2XscIU-KHWakY0bigvtpIlKazNHFvzFh8FRscDHg7MmkBFsv2j41ZmVsNNMdVKadZpw8mM-Kru0NlfDiD1M3MPlwdTyQRBFD3T2r_ZgctFU8gykaO9BqgfPQ-nQsGXzb3OKq1hX4HjDH8AgNvo2GcF6_KnzdO4-OZNvWkvKtCGxkWlG66HzQsRUzvCokhmhm0jCtXNC8TSgMJ6PKr6rK0oL8eXy9i11mO4dPm4uqV72OD-JWZMS8av0b72LCOztiN7nOjInYApDZAM8dKI_3A6IqBNoHLqUdeJ4rMbn2V7SubxE4vv1D7oUEJywpQhqdcNafg-qaMAeysKuAFt6B2X5lCbiPdRnYIlE7eZ3Va7Bu8ZodNqi5x6azCMO5sr45RvoYVq3gQHtf7cKvN5WbjuQyQ78OmQoFm0fBK97tCdP3_WyJiHqnDdaIPn9c9E3fSLj6b85Mn09s3zN0KYIQCLnbohZbcJg1vFVt2wDQXf0bjqZEKSuJo82qD8qb_Rakgz1DeDFQb4HTA7wMnRsc-bxt1v_0y1ng1LcQF0dSGxZYlgtFXRYkFTKDXD5mQxV_03SbQXtApkRJmf63lMbSE2ypTh6RFU77h1Raspw6P88sjHP1eRPv34tu2M2s3vFP9PH1o7enWxg7Whoh-rF4WZq2AYKmMP0KTXTdYWPgWijNoWWlYD_WhOFjz8v7fguTUFc-ZXGn5PY5tfx2leFpnSmyo5njhIb9cRnM2G7Ek9DvdOU3ukyUBMKY4_B2z8W5Qdop7SNSXk3oIebFdSMdyTFfYEjqaNsRXzRioXm8eC1iVyo4v2nlPEWaeCAPwQ2oH6mp_c_bCCpvIj5RBu76R7AKMV0PbsQxj83PNncya1sz234uBtzeh_BYTxfKO8-sOOqleBOkQFtbqeiJ_s7IdTL4cvKOAfEkA62AIaQ-Xw2Qdc_tsuQvTHdTo9qPtGXbTlzthl_YMwhBu5m00)
