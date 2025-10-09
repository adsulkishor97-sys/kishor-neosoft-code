public async Task<List<GetAssetListResponse>> GetAssetList(GetAssetListRequest request)
{
    // 1. Get base query
    var getAssetLists = new GetAssetListBigDataQuery();
    var assetListQuery = getAssetLists.assetListQueries.Select(x => x.query).FirstOrDefault();
    var query = GetAssetListReplaceQuery(assetListQuery!, request);

    // 2. Execute BigData query and map to GetAssetListResponse
    var bigDataResults = await _currentRepository.ExecuteBigDataQuery_New<GetAssetListResponse>(
        query!,
        MapReaderToAssetResponse
    ) ?? new List<GetAssetListResponse>();

    // 3. Get plantIds for secondary query
    var plantIds = string.Join(",", bigDataResults.Select(x => x.plantId).Distinct().Where(id => id > 0));
    var productDetails = await _benchMarkRepository.GetAssetListFromSqlDBAsync(plantIds!);

    // 4. Merge BigData results with ProductDetails
    var finalResult = bigDataResults.Select(b => MergeWithProductDetails(b, productDetails)).ToList();

    return finalResult;
}

// Helper: Map SQL reader to GetAssetListResponse
private GetAssetListResponse MapReaderToAssetResponse(IDataReader reader)
{
    return new GetAssetListResponse
    {
        assetId = reader["asset_id"]?.ToString() ?? "0",
        sapId = reader["sap_id"]?.ToString() ?? "0",
        assetName = reader["asset_name"]?.ToString() ?? "0",
        affiliateId = reader["affiliate_id"] != DBNull.Value ? Convert.ToInt32(reader["affiliate_id"]) : 0,
        affiliateName = reader["affiliate_name"]?.ToString() ?? "0",
        plantId = reader["plant_id"] != DBNull.Value ? Convert.ToInt32(reader["plant_id"]) : 0,
        plantName = reader["plant_name"]?.ToString() ?? "0",
        processServiceName = reader["process_service_name"]?.ToString() ?? "0",
        modelNumber = reader["model_number"]?.ToString() ?? "0",
        assetClassId = reader["asset_class_id"] != DBNull.Value ? Convert.ToInt32(reader["asset_class_id"]) : 0,
        designClassificationName = reader["design_classification_name"]?.ToString() ?? "0",
        companyName = reader["company_name"]?.ToString() ?? "0",
        assetClassName = reader["asset_class_name"]?.ToString() ?? "0",
    };
}

// Helper: Merge BigData result with ProductDetails
private GetAssetListResponse MergeWithProductDetails(GetAssetListResponse asset, List<ProductDetailsBySapIdSpResponse> productDetails)
{
    var product = productDetails.Find(p => p.plantId == asset.plantId);

    return new GetAssetListResponse
    {
        assetId = asset.assetId,
        sapId = asset.sapId?.Trim(),
        assetName = asset.assetName?.Trim(),
        affiliateId = asset.affiliateId,
        affiliateName = asset.affiliateName?.Trim(),
        productId = product?.productId ?? 0,
        plantId = asset.plantId,
        plantName = asset.plantName?.Trim(),
        processServiceName = asset.processServiceName?.Trim(),
        modelNumber = asset.modelNumber?.Trim(),
        assetClassId = asset.assetClassId,
        designClassificationName = asset.designClassificationName?.Trim(),
        companyName = asset.companyName?.Trim(),
        assetClassName = asset.assetClassName?.Trim()
    };
}
