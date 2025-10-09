public async Task<List<GetAssetListResponse>> GetAssetList(GetAssetListRequest request)
{
    // Step 1: Build Query
    var query = BuildAssetListQuery(request);

    // Step 2: Fetch BigData Results
    var bigDataResults = await FetchBigDataAssetListAsync(query);

    if (bigDataResults == null || bigDataResults.Count == 0)
        return new List<GetAssetListResponse>();

    // Step 3: Fetch SQL Results
    var productDetails = await FetchSqlAssetDetailsAsync(bigDataResults);

    // Step 4: Merge and Return
    return MergeAssetData(bigDataResults, productDetails);
}
#region Helper Methods

private string BuildAssetListQuery(GetAssetListRequest request)
{
    var assetQuery = new GetAssetListBigDataQuery()
        .assetListQueries
        .Select(x => x.query)
        .FirstOrDefault();

    if (string.IsNullOrEmpty(assetQuery))
        throw new InvalidOperationException("Asset list query not found in query configuration.");

    return GetAssetListReplaceQuery(assetQuery, request);
}

private async Task<List<GetAssetListResponse>> FetchBigDataAssetListAsync(string query)
{
    return await _currentRepository.ExecuteBigDataQuery_New<GetAssetListResponse>(
        query,
        reader => new GetAssetListResponse
        {
            assetId = SafeGetString(reader, "asset_id"),
            sapId = SafeGetString(reader, "sap_id"),
            assetName = SafeGetString(reader, "asset_name"),
            affiliateId = SafeGetInt(reader, "affiliate_id"),
            affiliateName = SafeGetString(reader, "affiliate_name"),
            plantId = SafeGetInt(reader, "plant_id"),
            plantName = SafeGetString(reader, "plant_name"),
            processServiceName = SafeGetString(reader, "process_service_name"),
            modelNumber = SafeGetString(reader, "model_number"),
            assetClassId = SafeGetInt(reader, "asset_class_id"),
            designClassificationName = SafeGetString(reader, "design_classification_name"),
            companyName = SafeGetString(reader, "company_name"),
            assetClassName = SafeGetString(reader, "asset_class_name")
        }) ?? new List<GetAssetListResponse>();
}

private async Task<List<ProductDetailsBySapIdSpResponse>> FetchSqlAssetDetailsAsync(List<GetAssetListResponse> bigDataResults)
{
    var plantIds = string.Join(",", bigDataResults
        .Select(x => x.plantId)
        .Distinct()
        .Where(id => id > 0));

    return await _benchMarkRepository.GetAssetListFromSqlDBAsync(plantIds);
}

private static List<GetAssetListResponse> MergeAssetData(
    List<GetAssetListResponse> bigDataResults,
    List<ProductDetailsBySapIdSpResponse> productDetails)
{
    return bigDataResults.Select(b =>
    {
        var product = productDetails.FirstOrDefault(p => p.plantId == b.plantId);

        return new GetAssetListResponse
        {
            assetId = b.assetId,
            sapId = b.sapId?.Trim(),
            assetName = b.assetName?.Trim(),
            affiliateId = b.affiliateId,
            affiliateName = b.affiliateName?.Trim(),
            productId = product?.productId ?? 0,
            plantId = b.plantId,
            plantName = b.plantName?.Trim(),
            processServiceName = b.processServiceName?.Trim(),
            modelNumber = b.modelNumber?.Trim(),
            assetClassId = b.assetClassId,
            designClassificationName = b.designClassificationName?.Trim(),
            companyName = b.companyName?.Trim(),
            assetClassName = b.assetClassName?.Trim()
        };
    }).ToList();
}

#endregion
#region Utility Safe Reader Helpers

private static string SafeGetString(IDataRecord reader, string columnName)
{
    try
    {
        var value = reader[columnName];
        return value != DBNull.Value ? value.ToString() ?? string.Empty : "0";
    }
    catch
    {
        return "0";
    }
}

private static int SafeGetInt(IDataRecord reader, string columnName)
{
    try
    {
        var value = reader[columnName];
        return value != DBNull.Value ? Convert.ToInt32(value) : 0;
    }
    catch
    {
        return 0;
    }
}

#endregion
