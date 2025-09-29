public async Task<List<GroupedData>> GetAffiliatesDistributionAsyncNew(GetAffiliatePlantDistributionRequest request, string affiliateRequest, int? plantId)
{
    var groupData = new List<GroupedData>();
    
    var (formattedStartDate, formattedEndDate) = FormatRequestDates(request);
    var sqlDataSubCategoryList = await _currentRepository.GetAssetClassAsync();
    var sqlKpiDataList = await _currentRepository.GetKPIDetailFromSqlDBAsync(request.kpiCode);
    
    var globalAffiliateDistributions = new GlobalAffiliateDistribution().ToList();
    var affiliatePageDistributions = new AffiliatePageDistribution();
    
    var (jsonFileGlobalResponseList, jsonFileAffiliateResponseList) = await LoadJsonFileData(globalAffiliateDistributions, affiliatePageDistributions);
    
    var (quotedpmcode, quatedditems) = BuildQuotedCodes(request, sqlDataSubCategoryList);
    
    if (request.page == "global")
    {
        groupData = await ProcessGlobalPage(request, formattedStartDate, formattedEndDate, quotedpmcode, jsonFileAffiliateResponseList);
    }
    else
    {
        groupData = await ProcessNonGlobalPage(request, formattedStartDate, formattedEndDate, quotedpmcode, jsonFileGlobalResponseList);
    }
    
    AssignTargetValues(groupData, sqlKpiDataList, request);
    
    return groupData;
}

private (string formattedStartDate, string formattedEndDate) FormatRequestDates(GetAffiliatePlantDistributionRequest request)
{
    var startDateTime = DateTime.Parse(request.startDate, CultureInfo.InvariantCulture);
    var endDateTime = DateTime.Parse(request.endDate, CultureInfo.InvariantCulture);
    
    return (startDateTime.ToString("yyyyMM"), endDateTime.ToString("yyyyMM"));
}

private async Task<(List<AffiliateDistribution> globalList, List<AffiliateDistribution> affiliateList)> LoadJsonFileData(
    List<AffiliateDistribution> globalDistributions, 
    AffiliatePageDistribution pageDistributions)
{
    var jsonFileGlobalResponseList = globalDistributions.affiliateDistributions.ToList();
    var jsonFileAffiliateResponseList = pageDistributions.affiliateDataList.ToList();
    
    return (jsonFileGlobalResponseList, jsonFileAffiliateResponseList);
}

private (string quotedpmcode, List<string> quatedditems) BuildQuotedCodes(
    GetAffiliatePlantDistributionRequest request, 
    List<AssetClass> sqlDataSubCategoryList)
{
    string quotedpmcode = "";
    
    if (Convert.ToInt32(request.kpiCode) == (int)KpiDetailConstants.mtbf || 
        Convert.ToInt32(request.kpiCode) == (int)KpiDetailConstants.mttr)
    {
        var pmCodes = sqlDataSubCategoryList
            .Where(x => x.assetClass == request.subCategory)
            .Select(x => x.pmCode)
            .FirstOrDefault();
        
        var items = pmCodes?.Trim('{', '}').Split(',') ?? Array.Empty<string>();
        var quatedditems = items.Select(x => $"'{x}'").ToList() ?? new List<string>();
        quotedpmcode = string.Join(",", quatedditems);
        
        return (quotedpmcode, quatedditems);
    }
    
    return ("", new List<string>());
}

private async Task<List<GroupedData>> ProcessGlobalPage(
    GetAffiliatePlantDistributionRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string quotedpmcode,
    List<AffiliateDistribution> jsonFileAffiliateResponseList)
{
    var lstAffiliates = await _currentRepository.GetAffiliateLists();
    var getAffiliateCodes = lstAffiliates
        .Where(a => a.affiliateId == request.affiliateId)
        .Select(x => x.affiliateCode)
        .ToList();
    
    var formatAffiliateCodes = $"'{string.Join("','", getAffiliateCodes.Select(n => $"'{n}'"))}'";
    
    var lstPlants = await _currentRepository.GetCaseHierarchyAsync(request.affiliateId);
    var getPlantIds = lstPlants.Select(x => x.plantId).ToList();
    var formatPlantIds = string.Join(",", getPlantIds);
    
    var formatAffiliateCodes2 = string.Join(",", getAffiliateCodes);
    
    var groupData = new List<GroupedData>();
    
    foreach (var item in jsonFileAffiliateResponseList.Where(a => a.kpiCode == request.kpiCode))
    {
        var numQuery = ReplaceQuery(item.overallN, formattedStartDate, formattedEndDate, formatAffiliateCodes, request, quotedpmcode, formatPlantIds);
        var overAllNumResult = await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(numQuery, reader => new KpiNumeratorDenominatorAffiliateDistribution
        {
            overallNumerator = reader["numerator"] != DBNull.Value ? Convert.ToDecimal(reader["numerator"]) : 0,
            kpiid = reader["kpi_id"] != DBNull.Value ? Convert.ToInt32(reader["kpi_id"]) : 0,
            plantid = reader["plantid"] != DBNull.Value ? Convert.ToInt32(reader["plantid"]) : 0,
            affiliates = reader["affiliateid"] != DBNull.Value ? Convert.ToInt32(reader["affiliateid"]) : 0
        });
        
        var denQuery = ReplaceQuery(item.overallD, formattedStartDate, formattedEndDate, formatAffiliateCodes, request, quotedpmcode, formatPlantIds);
        var overAllDenResult = await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(denQuery, reader => new KpiNumeratorDenominatorAffiliateDistribution
        {
            overallDenominator = reader["denominator"] != DBNull.Value ? Convert.ToDecimal(reader["denominator"]) : 0,
            kpiid = reader["kpi_id"] != DBNull.Value ? Convert.ToInt32(reader["kpi_id"]) : 0,
            plantid = reader["plantid"] != DBNull.Value ? Convert.ToInt32(reader["plantid"]) : 0,
            affiliates = reader["affiliateid"] != DBNull.Value ? Convert.ToInt32(reader["affiliateid"]) : 0
        });
        
        groupData = await GetAffDistFinalOverallNumDenPlantResult(overAllNumResult, overAllDenResult);
    }
    
    return groupData;
}

private async Task<List<GroupedData>> ProcessNonGlobalPage(
    GetAffiliatePlantDistributionRequest request,
    string formattedStartDate,
    string formattedEndDate,
    string quotedpmcode,
    List<AffiliateDistribution> jsonFileGlobalResponseList)
{
    string affiliateRequest1 = GetAffiliateRequestString();
    
    var groupData = new List<GroupedData>();
    
    foreach (var item in jsonFileGlobalResponseList.Where(a => a.kpiCode == request.kpiCode))
    {
        var current_kpiCode = Convert.ToInt32(item.kpiCode);
        
        if (IsMaintenanceCostKpi(current_kpiCode))
        {
            groupData = await ProcessMaintenanceCostQueries(item, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode);
        }
        else if (current_kpiCode == (int)KpiDetailConstants.availability)
        {
            groupData = await ProcessAvailabilityQuery(item, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode);
        }
        else
        {
            groupData = await ProcessOverallCriticalQuery(item, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode);
        }
    }
    
    return groupData;
}

private bool IsMaintenanceCostKpi(int kpiCode)
{
    return kpiCode == (int)KpiDetailConstants.reactiveMaintenanceCost ||
           kpiCode == (int)KpiDetailConstants.reactiveMaintenance ||
           kpiCode == (int)KpiDetailConstants.emergencyMaintenance ||
           kpiCode == (int)KpiDetailConstants.urgentMaintenance ||
           kpiCode == (int)KpiDetailConstants.pmRatio;
}

private async Task<List<GroupedData>> ProcessMaintenanceCostQueries(
    AffiliateDistribution item,
    string formattedStartDate,
    string formattedEndDate,
    string affiliateRequest1,
    GetAffiliatePlantDistributionRequest request,
    string quotedpmcode)
{
    var queryOverallN = ReplaceQuery(item.overallN, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode, "");
    var queryOverallD = ReplaceQuery(item.overallD, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode, "");
    var queryCriticalN = ReplaceQuery(item.criticalN, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode, "");
    var queryCriticalD = ReplaceQuery(item.criticalD, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode, "");
    
    var overAllNum = await ExecuteKpiQuery(queryOverallN);
    var overAllDen = await ExecuteKpiQuery(queryOverallD);
    var criticalNum = await ExecuteKpiQuery(queryCriticalN);
    var criticalDen = await ExecuteKpiQuery(queryCriticalD);
    
    return await GetAffDistFinalNumeratorDenominatorResult(overAllNum, overAllDen, criticalNum, criticalDen);
}

private async Task<List<GroupedData>> ProcessAvailabilityQuery(
    AffiliateDistribution item,
    string formattedStartDate,
    string formattedEndDate,
    string affiliateRequest1,
    GetAffiliatePlantDistributionRequest request,
    string quotedpmcode)
{
    var query = ReplaceQuery(item.overallN, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode, "");
    var overAllCriticalResult = await ExecuteKpiQuery(query);
    
    return await GetAffDistFinalOverallCriticalResult(overAllCriticalResult);
}

private async Task<List<GroupedData>> ProcessOverallCriticalQuery(
    AffiliateDistribution item,
    string formattedStartDate,
    string formattedEndDate,
    string affiliateRequest1,
    GetAffiliatePlantDistributionRequest request,
    string quotedpmcode)
{
    var query = ReplaceQuery(item.overallN, formattedStartDate, formattedEndDate, affiliateRequest1, request, quotedpmcode, "");
    var overAllCriticalResult = await ExecuteKpiQuery(query);
    
    return await GetAffDistFinalOverallCriticalResult(overAllCriticalResult);
}

private async Task<KpiNumeratorDenominatorAffiliateDistribution> ExecuteKpiQuery(string query)
{
    return await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(query, reader => new KpiNumeratorDenominatorAffiliateDistribution
    {
        overallNumerator = reader["oNum"] != DBNull.Value ? Convert.ToDecimal(reader["oNum"]) : 0,
        criticalNumerator = reader["critical"] != DBNull.Value ? Convert.ToDecimal(reader["critical"]) : 0,
        kpiid = reader["kpiid"] != DBNull.Value ? Convert.ToInt32(reader["kpiid"]) : 0,
        affiliates = reader["affiliateid"] != DBNull.Value ? Convert.ToInt32(reader["affiliateid"]) : 0
    });
}

private void AssignTargetValues(List<GroupedData> groupData, List<KpiData> sqlKpiDataList, GetAffiliatePlantDistributionRequest request)
{
    var targetMin = sqlKpiDataList
        .Where(x => x.kpiId == Convert.ToInt32(request.kpiCode))
        .Select(x => x.kpiMin)
        .FirstOrDefault();
    
    var targetMax = sqlKpiDataList
        .Where(x => x.kpiId == Convert.ToInt32(request.kpiCode))
        .Select(x => x.kpiMax)
        .FirstOrDefault();
    
    foreach (var itemData in groupData)
    {
        itemData.overallTarget = targetMax;
        
        if (itemData.overall >= targetMin && itemData.overall <= targetMax)
        {
            itemData.overallState = 1;
        }
        
        if (itemData.critical >= targetMin && itemData.critical <= targetMax)
        {
            itemData.criticalState = 0;
        }
    }
}

private string GetAffiliateRequestString()
{
    return "'1400','3200','3210','1600','3300','1500','2000','2020','2030','4000','3100','1700','3500','1900','1200','2200','1300','1800','3400'";
}

private string ReplaceQuery(string template, string startDate, string endDate, string affiliateCodes, GetAffiliatePlantDistributionRequest request, string quotedpmcode, string plantIds)
{
    return template
        .Replace("{startDate}", startDate)
        .Replace("{endDate}", endDate)
        .Replace("{affiliateCodes}", affiliateCodes)
        .Replace("{kpiCode}", request.kpiCode.ToString())
        .Replace("{quotedpmcode}", quotedpmcode)
        .Replace("{plantIds}", plantIds);
}

// Placeholder methods for result processing - implement based on your actual logic
private async Task<List<GroupedData>> GetAffDistFinalOverallNumDenPlantResult(
    KpiNumeratorDenominatorAffiliateDistribution numResult, 
    KpiNumeratorDenominatorAffiliateDistribution denResult)
{
    // Implementation here
    return new List<GroupedData>();
}

private async Task<List<GroupedData>> GetAffDistFinalNumeratorDenominatorResult(
    KpiNumeratorDenominatorAffiliateDistribution overAllNum,
    KpiNumeratorDenominatorAffiliateDistribution overAllDen,
    KpiNumeratorDenominatorAffiliateDistribution criticalNum,
    KpiNumeratorDenominatorAffiliateDistribution criticalDen)
{
    // Implementation here
    return new List<GroupedData>();
}

private async Task<List<GroupedData>> GetAffDistFinalOverallCriticalResult(
    KpiNumeratorDenominatorAffiliateDistribution result)
{
    // Implementation here
    return new List<GroupedData>();
}
