public async Task<List<AssetGroupedData>> GetAffDistFinalOverallNumAssetResult(List<KpiNumeratorDenominatorAffiliateDistribution> overAllNum, string category, string kpiCode)
{
    dynamic finalResult;
    var allplants = await _currentRepository.GetPlantLists();

    var getaffiliatenamelist = (from r in overAllNum
                                join plnt in allplants
                                on r.plantid equals plnt.plantId
                                select new
                                {
                                    r.kpiid,
                                    plantname = plnt.plantName,
                                    r.overallNumerator,
                                    r.sap_id,
                                    r.template,
                                    r.tidnr,
                                    r.pltxt,
                                    r.plantid
                                }
                      ).ToList();
    var sumOnlyKpis = new HashSet<string>() { "166" };
    bool isSumOnly = sumOnlyKpis.Contains(kpiCode);
    decimal? overallNume = null;
    if (category == "U")
    {
        finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.pltxt }).//sum up the multiple affiliate numerator and denominator and get the actual value.
           Select(g => 

           {
               bool hasValue = g.Any(x => x.overallNumerator != null);
               if (hasValue)
               {
                   if (isSumOnly)
                   {
                       overallNume = g.Sum(x => x.overallNumerator);
                   }
                   else
                   {
                       overallNume = g.Average(x => x.overallNumerator);
                   }
               }
               return new KpiNumDenAffiliateDistributionResult
               {
                   kpiid = g.Key.kpiid,
                   overallNum = overallNume,
                   plant = g.Select(x => x.plantid).FirstOrDefault(),
                   sapId = g.Select(x => x.sap_id).FirstOrDefault(),
                   template = g.Select(x => x.template).FirstOrDefault(),
                   tidnr = g.Select(x => x.tidnr).FirstOrDefault(),
                   pltxt = g.Select(x => x.pltxt).FirstOrDefault(),
               };
           }).OrderBy(x => x.kpiid).ThenBy(x => x.plantname).ToList();
    }
    else if (category == "AC")
    {
        finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.template }).//sum up the multiple affiliate numerator and denominator and get the actual value.
           Select(g => 

           {
               bool hasValue = g.Any(x => x.overallNumerator != null);
               if (hasValue)
               {
                   if (isSumOnly)
                   {
                       overallNume = g.Sum(x => x.overallNumerator);
                   }
                   else
                   {
                       overallNume = g.Average(x => x.overallNumerator);
                   }
               }
               return new KpiNumDenAffiliateDistributionResult
               {
                   kpiid = g.Key.kpiid,
                   overallNum = overallNume,
                   plant = g.Select(x => x.plantid).FirstOrDefault(),
                   sapId = g.Select(x => x.sap_id).FirstOrDefault(),
                   template = g.Select(x => x.template).FirstOrDefault(),
                   tidnr = g.Select(x => x.tidnr).FirstOrDefault(),
                   pltxt = g.Select(x => x.pltxt).FirstOrDefault(),
               };
           }).OrderBy(x => x.kpiid).ThenBy(x => x.plantname).ToList();
    }
    else
    {
        finalResult = getaffiliatenamelist.GroupBy(x => new { x.kpiid, x.tidnr }).//sum up the multiple affiliate numerator and denominator and get the actual value.
           Select(g => 

           {
               bool hasValue = g.Any(x => x.overallNumerator != null);
               if (hasValue)
               {
                   if (isSumOnly)
                   {
                       overallNume = g.Sum(x => x.overallNumerator);
                   }
                   else
                   {
                       overallNume = g.Average(x => x.overallNumerator);
                   }
               }
               return new KpiNumDenAffiliateDistributionResult
               {
                   kpiid = g.Key.kpiid,
                   overallNum = overallNume,
                   plant = g.Select(x => x.plantid).FirstOrDefault(),
                   sapId = g.Select(x => x.sap_id).FirstOrDefault(),
                   template = g.Select(x => x.template).FirstOrDefault(),
                   tidnr = g.Select(x => x.tidnr).FirstOrDefault(),
                   pltxt = g.Select(x => x.pltxt).FirstOrDefault(),
               };
           }).OrderBy(x => x.kpiid).ThenBy(x => x.plantname).ToList();
    }

    List<AssetGroupedData> lstResult = new List<AssetGroupedData>();
    foreach (var item in finalResult)
    {
        AssetGroupedData assetDistResult = new AssetGroupedData();
        assetDistResult.plantId = Convert.ToString(item.plant);
        assetDistResult.overall = item.overallNum;
        assetDistResult.sapId = item.sapId;
        assetDistResult.template = item.template;
        assetDistResult.tidnr = item.tidnr;
        assetDistResult.pltxt = item.pltxt;
        lstResult.Add(assetDistResult);
    }
    return lstResult;
}
