private async Task<List<AssetGroupedData>> GetOverAllNumeratorAsync(AffiliateDistribution item, string formatedStartDate, string formatedEndDate, string affiliateRequest, GetAffiliatePlantDistributionRequest requestRes, string quotedPmCodes, string plantId, string category)
{
    var numQuery = ReplaceQuery(item.overalN!, formatedStartDate, formatedEndDate, affiliateRequest, requestRes, quotedPmCodes, plantId);
    var overAllNumResult = await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(numQuery, reader => new KpiNumeratorDenominatorAffiliateDistribution
    {
        overallNumerator = reader["numerator"] != DBNull.Value ? Convert.ToDecimal(reader["numerator"]) : 0,
        kpiid = reader["kpi_id"] != DBNull.Value ? Convert.ToInt32(reader["kpi_id"]) : 0,
        sap_id = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"]) : "",
        template = reader["template"] != DBNull.Value ? Convert.ToString(reader["template"]) : "",
        tidnr = reader["tidnr"] != DBNull.Value ? Convert.ToString(reader["tidnr"]) : "",
        pltxt = reader["pltxt"] != DBNull.Value ? Convert.ToString(reader["pltxt"]) : "",
        plantid = int.TryParse(plantId, out var plant) ? plant : 0
    }) ?? new List<KpiNumeratorDenominatorAffiliateDistribution>();
    return await GetAffDistFinalOverallNumAssetResult(overAllNumResult, category);
}
calling method 
groupData = await GetOverAllNumeratorAsync(item, formatedStartDate, formatedEndDate, affiliateRequest, requestRes, quotedPmCodes, Convert.ToString(request.plantId)!, request.category);
