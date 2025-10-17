 private async Task<string> GetPmCodesAsync(KpiInputRequest request)
        {
            var sqlDataSubCategoryList = await _currentRepository.GetAssetClassAsync();
            var pscode = sqlDataSubCategoryList.Find(x => x.assetClass == request.subcategory)?.pmCode;
            if (string.IsNullOrEmpty(pscode)) return string.Empty;
            var quotedItems = pscode.Split(',').Select(x => $"'{x.Trim()}'");
            return string.Join(",", quotedItems);
        }
