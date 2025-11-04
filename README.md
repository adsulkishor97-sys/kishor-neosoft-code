private async Task<List<GroupedData>> HandleAffiliateDistributionGlobalPageRequest(GetAffiliatePlantDistributionRequest request, string affiliateRequest, string formatedStartDate, string formatedEndDate, string quotedPmCodes)
{
    var globalAffiliateDistributions = new GlobalAffiliateDistribution();
    List<AffiliateDistribution> jsonFileGlobalResponseList = globalAffiliateDistributions.affiliateDistributions.ToList();
    var item = jsonFileGlobalResponseList.Find(a => a.kpiCode == request.kpiCode) ?? new AffiliateDistribution();
    var current_kpiCode = Convert.ToInt32(item.kpiCode);
    if (IsReactiveOrMaintenanceKpi(current_kpiCode))
        return await HandleReactiveAndMaintenanceKpi(item, affiliateRequest, request, formatedStartDate, formatedEndDate, quotedPmCodes);
    if (IsAvailabilityKpi(current_kpiCode))
        return await HandleAvailabilityKpi(item, affiliateRequest, request, formatedStartDate, formatedEndDate, quotedPmCodes);
    return await HandleDefaultKpi(item, affiliateRequest, request, formatedStartDate, formatedEndDate, quotedPmCodes);
}
