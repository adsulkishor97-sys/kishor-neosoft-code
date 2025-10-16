public async Task<List<GroupedData>> GetAffiliatesDistributionAsyncNew(GetAffiliatePlantDistributionRequest request, string affiliateRequest, int? plantId)
{
    var dateresponse = ParseAndFormatDates(request.startDate!, request.endDate!);
    var formatedStartDate = dateresponse.startDateTimeDate;
    var formatedEndDate = dateresponse.endDateTimeDate;
    var sqlDataSubCategoryList = await _currentRepository.GetAssetClassAsync();
    var sqlKpiDataList = await _currentRepository.GetKPIDetailFromSqlDBAsync(request.kpiCode!);
    var quotedPmCodes = GetPMCodes(request, sqlDataSubCategoryList);
    var groupData = await ProcessAffiliatesDistributionAsync(request, affiliateRequest, formatedStartDate, formatedEndDate, quotedPmCodes);
    groupData = GetFinalGroupData(sqlKpiDataList, groupData, request.kpiCode!);
    return groupData;
}
private async Task<List<GroupedData>> ProcessAffiliatesDistributionAsync(GetAffiliatePlantDistributionRequest request,string affiliateRequest, string formatedStartDate, string formatedEndDate, string quotedPmCodes)
{
    if (request.page != "global")
    {
        return await HandleAffiliateDistributionNonGlobalPageRequest(request, affiliateRequest, formatedStartDate, formatedEndDate, quotedPmCodes);
    }
    else
    {
        return await HandleAffiliateDistributionGlobalPageRequest(request, affiliateRequest, formatedStartDate, formatedEndDate, quotedPmCodes);
    }
}
private async Task<List<GroupedData>> HandleAffiliateDistributionNonGlobalPageRequest(GetAffiliatePlantDistributionRequest request,string affiliateRequest, string formatedStartDate, string formatedEndDate, string quotedPmCodes)
{
    AffiliatePageDistribution affiliatePageDistributions = new AffiliatePageDistribution();
    List<AffiliateDistribution> jsonFileAffiliateResponseList = affiliatePageDistributions.affiliateDataList.ToList();
    var lstPlants = await _currentRepository.GetCaseHierarchyAsync(request.affiliateId!) ?? new List<GetCaseHierarchyResponse>();

    var formatAffilateCodes = affiliateRequest;
    var formatPlantIds = GetPlantIds(lstPlants);
    var item = jsonFileAffiliateResponseList.Find(a => a.kpiCode == request.kpiCode) ?? new AffiliateDistribution();

    var numQuery = ReplaceQuery(item.overalN!, formatedStartDate, formatedEndDate, formatAffilateCodes, request, quotedPmCodes, formatPlantIds);
    var overAllNumResult = await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(numQuery, reader => new KpiNumeratorDenominatorAffiliateDistribution
    {
        overallNumerator = reader["numerator"] != DBNull.Value ? Convert.ToDecimal(reader["numerator"]) : 0,
        kpiid = reader["kpi_id"] != DBNull.Value ? Convert.ToInt32(reader["kpi_id"]) : 0,
        plantid = reader["plantid"] != DBNull.Value ? Convert.ToInt32(reader["plantid"]) : 0,
    }) ?? new List<KpiNumeratorDenominatorAffiliateDistribution>();
    var denQuery = ReplaceQuery(item.overalD!, formatedStartDate, formatedEndDate, formatAffilateCodes, request, quotedPmCodes, formatPlantIds);
    var overAllDenResult = await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(denQuery, reader => new KpiNumeratorDenominatorAffiliateDistribution
    {
        overallDenominator = reader["denominator"] != DBNull.Value ? Convert.ToDecimal(reader["denominator"]) : 0,
        kpiid = reader["kpi_id"] != DBNull.Value ? Convert.ToInt32(reader["kpi_id"]) : 0,
        plantid = reader["plantid"] != DBNull.Value ? Convert.ToInt32(reader["plantid"]) : 0,

    }) ?? new List<KpiNumeratorDenominatorAffiliateDistribution>();
    return await GetAffDistFinalOverallNumDenPlantResult(overAllNumResult, overAllDenResult);
}
