public async Task<List<AssetGroupedData>> GetAffDistFinalOverallNumAssetResult(
    List<KpiNumeratorDenominatorAffiliateDistribution> overAllNum,
    string category,
    string kpiCode)
{
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
                                }).ToList();

    var sumOnlyKpis = new HashSet<string>() { "166" };
    bool isSumOnly = sumOnlyKpis.Contains(kpiCode);

    Func<dynamic, object> groupSelector = category switch
    {
        "U" => x => new { x.kpiid, x.pltxt },
        "AC" => x => new { x.kpiid, x.template },
        _ => x => new { x.kpiid, x.tidnr }
    };

    var finalResult = getaffiliatenamelist
        .GroupBy(groupSelector)
        .Select(g =>
        {
            decimal? overallNume = null;

            if (g.Any(x => x.overallNumerator != null))
            {
                overallNume = isSumOnly
                    ? g.Sum(x => x.overallNumerator)
                    : g.Average(x => x.overallNumerator);
            }

            var first = g.First();

            return new KpiNumDenAffiliateDistributionResult
            {
                kpiid = first.kpiid,
                overallNum = overallNume,
                plant = first.plantid,
                sapId = first.sap_id,
                template = first.template,
                tidnr = first.tidnr,
                pltxt = first.pltxt
            };
        })
        .OrderBy(x => x.kpiid)
        .ThenBy(x => x.plantname)
        .ToList();

    return finalResult.Select(item => new AssetGroupedData
    {
        plantId = Convert.ToString(item.plant),
        overall = item.overallNum,
        sapId = item.sapId,
        template = item.template,
        tidnr = item.tidnr,
        pltxt = item.pltxt
    }).ToList();
}
