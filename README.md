 private async Task<List<GroupedData>> HandleAvailabilityKpi(AffiliateDistribution item, string affiliateRequest1, GetAffiliatePlantDistributionRequest request, string formatedStartDate, string formatedEndDate, string quotedPmCodes)
 {
     var query = ReplaceQuery(item.overalN!, formatedStartDate, formatedEndDate, affiliateRequest1, request, quotedPmCodes, "");
     var overAllCriticalResult = await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(query, reader => new KpiNumeratorDenominatorAffiliateDistribution
     {
         overallNumerator = reader["overall"] != DBNull.Value ? Convert.ToDecimal(reader["overall"]) : 0,
         criticalNumerator = reader["critical"] != DBNull.Value ? Convert.ToDecimal(reader["critical"]) : 0,
         kpiid = reader["kpiid"] != DBNull.Value ? Convert.ToInt32(reader["kpiid"]) : 0,
         affiliates = reader["affiliateid"] != DBNull.Value ? Convert.ToInt32(reader["affiliateid"]) : 0
     }) ?? new List<KpiNumeratorDenominatorAffiliateDistribution>();
     return await GetAffDistFinalOverallCriticalResult(overAllCriticalResult);
 }
