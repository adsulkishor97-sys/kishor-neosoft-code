 public async Task<List<GetBenchmarkCriteriaList>> GetAffiliateCriteria(GetBenchmarkCriteriaListRequest request)
 {
     return await ExecuteQueryListAsync<GetBenchmarkCriteriaList>(
         SPConstant.USP_UI_GET_BENCHMARK_CRITERIA,
        new
        {
            page = request.page
        });
 }
 public async Task<List<AffiliateBenchmarkComparisionResponse>> GetAffiliateBenchmarkComparisionAsync(AffiliateBenchmarkComparisionRequest affiliateBenchmarkComparisionRequest)
{
    AffiliateBenchmarkComparisionQueries affiliatePageDistributions = new AffiliateBenchmarkComparisionQueries();

    List<BenchmarkComparision> affiliateResponseList = affiliatePageDistributions.affiliateDataList.ToList();

    // get AffilateCodes based on affilateId           

    var affiliateList = await _benchMarkRepository.GetAffiliateLists() ?? new List<AffiliateList>();
    var affiliateIds = affiliateBenchmarkComparisionRequest.affiliateId?.Split(',').Select(int.Parse).ToList();
    var formatAffilateCodes = string.Join(",", affiliateIds!);

    var tasks = affiliateResponseList
     .Where(a => a.kpiCode == affiliateBenchmarkComparisionRequest.kpiCode)
     .Select(async item =>
     {
         var query = ReplaceQuery(item.query!, affiliateBenchmarkComparisionRequest.startDate!, affiliateBenchmarkComparisionRequest.endDate!, formatAffilateCodes, "", "", "");

         var result = await _benchMarkRepository.ExecuteBigDataQueryAsync<AffiliateBenchmarkComparisionResponse>(query, reader => new AffiliateBenchmarkComparisionResponse
         {
             affiliateName = reader["affiliate_id"] != DBNull.Value ? affiliateList.Where(x => x.affiliateId == Convert.ToInt32(reader["affiliate_id"]!))
                             .Select(x => x.affiliateName).FirstOrDefault() : null,
             actual = reader["actual"] != DBNull.Value ? Convert.ToDecimal(reader["actual"]) : 0,
             absolute = reader["absolute"] != DBNull.Value ? Convert.ToDecimal(reader["absolute"]) : 0
         }) ?? new List<AffiliateBenchmarkComparisionResponse>();
         return result;
     }).ToList();
    var data = (await Task.WhenAll(tasks!)).ToList();
    var finalResult = data.SelectMany(x => x).ToList();
    return finalResult!;
}
