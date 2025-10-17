 private static decimal GetDecimalValue(dynamic reader, string column)
        {
            return reader[column] != DBNull.Value ? Convert.ToDecimal(reader[column]) : 0;
        }

         private static string GetPMCodes(GetAffiliatePlantDistributionRequest request, List<AssetClassSubCategory> sqlDataSubCategoryList)
        {
            if (!int.TryParse(request.kpiCode, out var kpiCode))
                return "''";
            if (kpiCode != (int)KpiDetailConstants.mtbf && kpiCode != (int)KpiDetailConstants.mttr)
                return "''";
            var pmCodes = sqlDataSubCategoryList
                .Find(x => x.assetClass == request.subCategory)?.pmCode;
            var codes = pmCodes?.Split(',').Select(x => $"'{x.Trim()}'").ToList() ?? new List<string>();
            return string.Join(",", codes);
        }
