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
