public async Task<List<GetAssetListResponse>> GetAssetList(GetAssetListRequest getAssetListRequest)
{
    List<GetAssetListResponse> getAssetLIstResponses;

    GetAssetListBigDataQuery getAssetLists = new GetAssetListBigDataQuery();

    var assetListQuery = getAssetLists.assetListQueries.Select(x => x.query).FirstOrDefault();
    var query=GetAssetListReplaceQuery(assetListQuery!.ToString(), getAssetListRequest);

    List<GetAssetListResponse> bigdataQueryeResult; 
    List<ProductDetailsBySapIdSpResponse> productDetailsBySapIdSpResponses = new List<ProductDetailsBySapIdSpResponse>();
   
        bigdataQueryeResult = await _currentRepository.ExecuteBigDataQuery_New<GetAssetListResponse>(query!, reader => new GetAssetListResponse
        {
            assetId = reader["asset_id"] != DBNull.Value ? reader["asset_id"].ToString() : "0",
            sapId = reader["sap_id"] != DBNull.Value ? reader["sap_id"].ToString() : "0",
            assetName= reader["asset_name"] != DBNull.Value ? reader["asset_name"].ToString() : "0",
            affiliateId = reader["affiliate_id"] != DBNull.Value ? Convert.ToInt32(reader["affiliate_id"]) : 0,
            affiliateName = reader["affiliate_name"] != DBNull.Value ? reader["affiliate_name"].ToString() : "0",
            plantId= reader["plant_id"] != DBNull.Value ? Convert.ToInt32(reader["plant_id"]) : 0,
            plantName = reader["plant_name"] != DBNull.Value ? reader["plant_name"].ToString() : "0",
            processServiceName = reader["process_service_name"] != DBNull.Value ? reader["process_service_name"].ToString() : "0",
            modelNumber= reader["model_number"] != DBNull.Value ? reader["model_number"].ToString() : "0",
            assetClassId= reader["asset_class_id"] != DBNull.Value ? Convert.ToInt32(reader["asset_class_id"]) :0,
            designClassificationName= reader["design_classification_name"] != DBNull.Value ? reader["design_classification_name"].ToString() : "0",
            companyName= reader["company_name"] != DBNull.Value ? reader["company_name"].ToString() : "0",
            assetClassName= reader["asset_class_name"] != DBNull.Value ? reader["asset_class_name"].ToString() : "0",



        }) ?? new List<GetAssetListResponse>();



     
        var plantIds = string.Join(",", bigdataQueryeResult!.Select(x => x.plantId).Distinct().Where(plantID => plantID > 0));
        productDetailsBySapIdSpResponses = await _benchMarkRepository.GetAssetListFromSqlDBAsync(plantIds!);

   

    getAssetLIstResponses = bigdataQueryeResult.Select(b =>
    {

        var productDetails = productDetailsBySapIdSpResponses.Find(p => p.plantId == b.plantId);
        return new GetAssetListResponse
        {
            assetId = b.assetId,
            sapId = b.sapId?.Trim(),
            assetName = b.assetName?.Trim(),
            affiliateId = b.affiliateId,
            affiliateName = b.affiliateName?.Trim(),
            productId = productDetails?.productId ?? 0,
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
   
    return getAssetLIstResponses!;
}
