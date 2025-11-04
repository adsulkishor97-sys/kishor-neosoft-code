 private async Task<List<GroupedData>> ProcessAffiliatesDistributionAsync(GetAffiliatePlantDistributionRequest request, string affiliateRequest, string formatedStartDate, string formatedEndDate, string quotedPmCodes)
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
