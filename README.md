 public async Task<List<AssetDesignSubgroupBySapId>> GetAssetDesignSubgroupBySapIds(DesignDataBySapIdRequest designDataBySapIdRequest)
 {
     var performanceKpires = await _assetRepository.GetAssetDesignSubgroupBySapIds(designDataBySapIdRequest);

     var finalResponse = performanceKpires.
         GroupBy(x => new { x.categoryId, x.categoryName })
         .Select(g => new AssetDesignSubgroupBySapId
         {
             categoryId = g.Key.categoryId,
             categoryName = g.Key.categoryName,
             assetDesignSubgroupDataItems = g.Select(i => new AssetDesignSubgroupDataItem
             {
                 mark = i.mark,
                 quantity = i.quantity,
                 size = i.size,
                 rating = i.rating,
                 service = i.service

             }).ToList()
         }).ToList();


     return finalResponse;
 }
 public class DesignDataBySapIdRequest
{
    
        public string? sapId { get; set; }
    
}
public  class AssetDesignSubgroupBySapId
{
    public string? categoryId { get; set; }
    public string? categoryName { get; set; }
    [JsonPropertyName("data")]
    public  List<AssetDesignSubgroupDataItem>? assetDesignSubgroupDataItems { get; set; }


}
public class AssetDesignSubgroupDataItem
{
    public string? mark { get; set; }
    public string? quantity { get; set; }
    public string? size { get; set; }
    public string? rating { get; set; }
    public string? service { get; set; }
}
