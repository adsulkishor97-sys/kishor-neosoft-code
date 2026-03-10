public async Task<List<AssetInfoResp>> GetAssetInfoAsync(AssetInfoRequest assetInfoRequest)
{
    List<AssetInfoResp> assetInformationBySapId = await _cacheService.GetList<AssetInfoRequest, AssetInfoResp>(assetInfoRequest, "get_asset_information_by_sap_id");
    if (assetInformationBySapId != null) return assetInformationBySapId;
    AssetPageQueries assetPageQueries = new AssetPageQueries();
    List<AssetCommon> assetCommons = assetPageQueries.assetInformation!.ToList();
    var query = assetCommons[0].query!.ToString();
    var queryString = query!.Replace("${sapId}", $"'{assetInfoRequest.sapID?.SapIdPrefix()}'");
    var bigDataSwatch = new Stopwatch();
    bigDataSwatch.Start();
    var result = await ExecuteBigDataQueryAsync<AssetInfoResp>(queryString, reader => new AssetInfoResp
    {

        assetId = reader["asset_id"] != DBNull.Value ? Convert.ToString(reader["asset_id"]) : "",
        sapId = reader["sap_id"] != DBNull.Value ? Convert.ToString(reader["sap_id"]) : "",
        assetDescription = reader["description"] != DBNull.Value ? Convert.ToString(reader["description"]) : "",
        functionalLocation = reader["functional_location"] != DBNull.Value ? Convert.ToString(reader["functional_location"]) : "",
        functionalDescription = reader["functional_description"] != DBNull.Value ? Convert.ToString(reader["functional_description"]) : "",
        model = reader["model"] != DBNull.Value ? Convert.ToString(reader["model"]) : "",
        manufacturer = reader["manufacturer"] != DBNull.Value ? Convert.ToString(reader["manufacturer"]) : "",
        assetCriticality = reader["asset_criticality"] != DBNull.Value ? Convert.ToString(reader["asset_criticality"]) : "",
        assetAcquisitionDateEpoch = reader["asset_acquisitiondate"] != DBNull.Value ? Convert.ToInt64(reader["asset_acquisitiondate"]) : 0,

        lastFailureDateEpoch = reader["last_failure_date"] != DBNull.Value ? Convert.ToInt64(reader["last_failure_date"]) : 0,

        upcomingPMDueDateEpoch = reader["upcoming_pm_due_date"] != DBNull.Value ? Convert.ToInt64(reader["upcoming_pm_due_date"]) : 0,
        ehssCritical = reader["ehss_critical"] != DBNull.Value ? Convert.ToString(reader["ehss_critical"]) : "",
        designClassification = reader["design_classification"] != DBNull.Value ? Convert.ToString(reader["design_classification"]) : "",
        standByAvailable = reader["standby_available"] != DBNull.Value ? Convert.ToString(reader["standby_available"]) : "",
        processService = reader["process_service"] != DBNull.Value ? Convert.ToString(reader["process_service"]) : "",
        assetType = reader["asset_type"] != DBNull.Value ? Convert.ToString(reader["asset_type"]) : "",
        lastoverhaulingDate = reader["overhauling"] != DBNull.Value ? Convert.ToInt64(reader["overhauling"]) : 0,
        lastBreakDownOnEpoch = reader["lastbreakdowndate"] != DBNull.Value ? Convert.ToInt64(reader["lastbreakdowndate"]) : 0,
        lastPreventiveMaintenanceOnEpoch = reader["lastpreventivemaintenanceon"] != DBNull.Value ? Convert.ToInt64(reader["lastpreventivemaintenanceon"]) : 0

    }) ?? new List<AssetInfoResp>();
    bigDataSwatch.Stop();
    await _cacheService.SetList<AssetInfoRequest, AssetInfoResp>(assetInfoRequest, result, keyPrefix: "get_asset_information_by_sap_id");
    return result!;
}
