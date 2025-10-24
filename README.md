
--//=====================================Plant Tab APIs=======================
{
  1) endpoint: "get_plant_filters",
  response: [
    {
      "filterName": "Affiliate",
      "data": [
        {
          "affiliateId": 11,
          "affiliateName": "AFF"
        },
        {
          "affiliateId": 123,
          "affiliateName": "ABB"
        },
        {
          "affiliateId": 1234,
          "affiliateName": "ACCC"
        }
      ]
    },
    {
    filterName: "Products",
    data: [
        {
            productId: 11,
            productName: "LLDE"
        },
        {
            productId: 1234,
            productName: "NHMK"
        },
        {
            productId: 567,
            productName: "PKJU"
        }
    ]
}

2)endpoint : get_plant_list
resp [
    {
        plantId: 1,
        plantName: "plantname01",
        affilateId: 11,
        productId: 11,
        productName: "PKJU",
        affiliateName: "ACCC"
                
    },
    {
        plantId: 2,
        plantName: "plantname02",
        affilateId: 123,
        productId: 1234,
        productName: "PKJU",
        affiliateName: "ACCC"
    },
{
        plantId: 3,
        plantName: "plantname03",
        affilateId: 1234,
        productId: 567,
        productName: "PKJU",
        affiliateName: "ACCC"
}
]

3) apiname: get_plant_benchmark
payload: {
    plantId: ["123, 124"], // comma separated affiliate id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025",
    assetClassId: 123
}
Response: [
    {
        
        plantId:"",        
        target: 100,
        direction: 0/1, // 0 => primary_blue | 1 => primary_yellow
        actual: 29,
        absolute: 29,
        bestAchievedEver: 95,
        bestAchievedEverMin: 95,
        bestAchievedForSinglePeriod: 15,
    }
]

4) endpoint : get_asset_class
response :[
    {
        assetClassId: 1,
        assetClassName: ""
    },
    {
        assetClassId: 1,
        assetClassName: ""
    }
]

5) apiname: get_plant_benchmark_comparision
payload: {
    plantId: "123, 124", // comma separated plant id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025",
    assetClassId: 123
}
Response: [
    {
        
        plantId:"",                  
        actual: 100,
        absolute: 200
    },
]

6) apiname: get_plant_trend
payload: {
    plantId: "123, 124", // comma separated affiliate id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025"
    assetClassId: 123
}
Response: [
    {
        
        plantId:"",             
        actual: 100,
        absolute: 45,
        time: 1234567890000,
        frequency:"month" | "quater"       
    }
]

======= common api ====
7) apiname: get_benchmark_criteria
payload: {
    page: affiliate/plant/asset
}
Response: [
    {
        kpiType: "Business KPI",
        data: [{
            kpiCode: "",
            kpiName: "",
            Unit: ""
        }]
    },
    {
        kpiType: "Performance KPI",
        data: [{
            kpiCode: "",
            kpiName: "",
            Unit: ""
        }]
    },
{
kpiType: "Performance KPI",
data: [{
    kpiCode: "",
    kpiName: "",
    Unit: ""
}]
}
]
------------------------------------

################# AFFILIATE TAB #################
apiname: get_affiliate_list // SQL
payload: 
Response: 
[
    {
        affiliateId: 001,
        affiliateName: "AFF"
    }
]

apiname: get_affiliate_criteria // SQL
payload: 
Response: [
    {
        kpiType: "Business KPI",
        data: [{
            kpiCode: "",
            kpiName: "",
        }]
    }
 
]


apiname: get_affiliate_benchmark
payload: {
    affiliateId: "123, 124",// comma separated affiliate id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025",

}
Response: [
    {
        affiliateName: "Yanpet",
        target: 100,
        state: 0/1, // 0 => primary_blue | 1 => primary_yellow
        direction: 1/2
        actual: 29,
        absolute: 29,
        bestAchievedEver: 95,
        bestAchievedEverMin: 95,
        bestAchievedForSinglePeriod: 15,


// get direction of the kpi from SQL  1- lower the better 
1 - lower the better
bestAchievedEverMin = min(all bestAchievedEver from all affiliates),
bestAchievedForSinglePeriod = min(all actual from all affilaites)

2/0 - lower the better
bestAchievedEverMin = max(all bestAchievedEver from all affiliates),
bestAchievedForSinglePeriod = max(all actual from all affilaites)   //


    }
]

apiname: get_affiliate_benchmark_comparision
payload: {
    affiliateId: "123, 124",// comma separated affiliate id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025",

}
Response: [
    {
        affiliateName: "Yanpet",
        actual: 100,
        absolute: 200
    },
]

apiname: get_affiliate_trend
payload: {
    affiliateId: ["123, 124"],// comma separated affiliate id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025"    
}
Response: [
    {
        affiliateName: "Yanpet",
        actual: 100,
        absolute: 123421,
        time: 202501,
        frequency:"month" | "quater"             

    }
]

---------------------------------------


apiname: get_asset_filters
payload:
response: [
    {
        filterName: "Affiliate",
        data: [
            {
                affiliateId: 000,
                affiliateName: "AFF"
            }
        ]
    },
    {
        filterName: "Products",
        data: [
            {
                productId: 000,
                productName: "LLDE"
            }
        ]
    },
{
        filterName: "Plant",
        data: [
            {
                plantId: 000,
                plantName: "LLDE"
            }
        ]
    },
{
        filterName: "Process_Services",
        data: [
            {
              
                processServiceName: "LLDE"
            }
        ]
    },
{
        filterName: "Asset Class",
        data: [
            {
                assetClassId: 000,
                assetClassName: "LLDE"
            }
        ]
    },
{
        filterName: "Design Classifictaion",
        data: [
            {
               
                designClassificationName: "LLDE"
            }
        ]
    },

{
        filterName: "Model Number",
        data: [
            {
               modelNumber: "LLDE"
            }
        ]
    },
{
        filterName: "Asset Manufacture",
        data: [
            {
              
                companyName: "LLDE"
            }
        ]
    },
]

========================

apiname: get_asset_list
payload: {
          affiliateId: [1,2,3], --nullable
          productId :[4,5,6],--nullable
          plantId: [7,8,9],  --nullable
          processServiceName: ["LLDE","LDPE"]  --nullable
          assetClassId: [10,15,16],--nullable
          designClassificationName: ["LLDE","LDPE"] --nullable
          modelNumber: ["LLDE","HDPE"]--nullable
          companyName: ["LLDE","HDPE"]    --nullable                  
          }
response: [
    {
        assetId: DT-000,
        sapId:"877676"
        assetName: "asset title",
        affilateId: 01,
        productId: 01,
        plantId: 01,
        processServiceName: "",
        modelNumber: 01,
        assetClassId:01,
        designClassificationName:"",
        companyName: "LLDE",
        assetClassName:""
    }
]


apiname: get_asset_benchmark
payload: {
    sapId: ["123, 124"],// comma separated asset id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025",
    }
Response: [
    {
    sapId:""
        target: 100,
        direction: 0/1, // 0 => primary_blue | 1 => primary_yellow
        actual: 29,
        absolute: 29,
        bestAchievedEver: 95,
        bestAchievedEverMin: 95,
        bestAchievedForSinglePeriod: 15,
      modelNumber: "",
      manufacturer: "",
      designValue:""
    }
]



apiname: get_asset_benchmark_comparision
payload: {
    sapId: ["123, 124"],// comma separated asset id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025",

}
Response: [
    {

        sapId:""
        actual: 29,
        absolute: 29,
    modelNumber: "",
    companyName: "",
    },
]



apiname: get_asset_trend
payload: {
    sapId: ["123, 124"],// comma separated affiliate id's
    kpiCode: 123,
    startDate: "22-12-2025",
    endDate: "22-12-2025", 
}
Response: [
    {

        sapId:""
        actual: 100,
        absolute:7678,
        time: 1234567890000,
        frequency: "month" | "quarter"
    }
]
    
]


apiname: get_asset_performance_criteria
payload: {
    sapId: ["123, 124"],// comma separated affiliate id's    
}
reponse: [
    {
        kpiType: "Business KPI",
        data: [{
            kpiCode: "",
            kpiName: "",
        }]
    }

]

MS: AFSDK
apiname: get_asset_benchmark_performance
payload: {
  sapId: ["123,124"],
  kpiCode: 312,
  startDate: "01-01-2025",
  endDate: "2025-09-01"      
}

Response: [
{
  sapId: "",
  target: null,
  direction: 0,
  actual: 34.22, //pidata pull
  absolute: null,
  bestAchievedEver: null,
  bestAchievedEverMin: null,
  bestAchievedForSinglePeriod: null,
  modelNumber: "",
  manufacturer: "",
  designValue:"" //map from design from SP      
}]


 apiname: get_asset_benchmark_performance
payload: {
  sapId: ["123,124"],
  kpiCode: 312      
}

Response: [
{
  sapId: "",
  target: null,
  direction: 0,
  actual: 34.22, //from SP
  absolute: null,
  bestAchievedEver: null,
  bestAchievedEverMin: null,
  bestAchievedForSinglePeriod: null,
  modelNumber: "", //from SP
  manufacturer: "",  //from SP
  designValue:null      
}]

 



