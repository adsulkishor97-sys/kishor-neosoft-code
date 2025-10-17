private static string GetQuotedPmCodesIfNeeded(GetAssetDistributionRequest request, List<AssetClassSubCategory> sqlDataSubCategoryList)
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
