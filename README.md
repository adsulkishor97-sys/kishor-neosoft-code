public class OverAllNumeratorRequest
{
    public AffiliateDistribution Item { get; set; }
    public string FormattedStartDate { get; set; } = string.Empty;
    public string FormattedEndDate { get; set; } = string.Empty;
    public string AffiliateRequest { get; set; } = string.Empty;
    public GetAffiliatePlantDistributionRequest RequestRes { get; set; } = new();
    public string QuotedPmCodes { get; set; } = string.Empty;
    public string PlantId { get; set; } = string.Empty;
    public string Category { get; set; } = string.Empty;
}
private async Task<List<AssetGroupedData>> GetOverAllNumeratorAsync(OverAllNumeratorRequest req)
{
    var numQuery = ReplaceQuery(req.Item.overalN!, req.FormattedStartDate, req.FormattedEndDate,
                                req.AffiliateRequest, req.RequestRes, req.QuotedPmCodes, req.PlantId);

    var overAllNumResult = await _currentRepository.ExecuteBigDataQuery_New<KpiNumeratorDenominatorAffiliateDistribution>(
        numQuery,
        reader => new KpiNumeratorDenominatorAffiliateDistribution
        {
            overallNumerator = reader["numerator"] != DBNull.Value ? Convert.ToDecimal(reader["numerator"]) : 0,
            kpiid = reader["kpi_id"] != DBNull.Value ? Convert.ToInt32(reader["kpi_id"]) : 0,
            sap_id = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"]) : "",
            template = reader["template"] != DBNull.Value ? Convert.ToString(reader["template"]) : "",
            tidnr = reader["tidnr"] != DBNull.Value ? Convert.ToString(reader["tidnr"]) : "",
            pltxt = reader["pltxt"] != DBNull.Value ? Convert.ToString(reader["pltxt"]) : "",
            plantid = int.TryParse(req.PlantId, out var plant) ? plant : 0
        }) ?? new List<KpiNumeratorDenominatorAffiliateDistribution>();

    return await GetAffDistFinalOverallNumAssetResult(overAllNumResult, req.Category);
}
var numeratorRequest = new OverAllNumeratorRequest
{
    Item = item,
    FormattedStartDate = formatedStartDate,
    FormattedEndDate = formatedEndDate,
    AffiliateRequest = affiliateRequest,
    RequestRes = requestRes,
    QuotedPmCodes = quotedPmCodes,
    PlantId = Convert.ToString(request.plantId)!,
    Category = request.category
};

groupData = await GetOverAllNumeratorAsync(numeratorRequest);
