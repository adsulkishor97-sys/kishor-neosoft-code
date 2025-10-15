private static async Task<KpiDetail> GenerateMaintenancePlantKpiReport(string kpiName, List<GetCaseHierarchyResponse> plantDetails, Dictionary<int, decimal> actuals, decimal targetValue)
{
    return await Task.Run(() =>
    {
        var data = (from plants in plantDetails
                    where actuals.ContainsKey((int)plants.plantId!)
                    let actualValue = actuals.ContainsKey((int)plants!.plantId!) ? actuals[(int)plants.plantId!] : 0
                    let state = actualValue > targetValue ? 1 : 0
                    select new Plant
                    {
                        plantName = plants.plantName,
                        actual = actualValue,
                        target = targetValue,
                        state = state
                    }).ToList();



        return new KpiDetail
        {
            kpi = kpiName,
            plants = data.OrderBy(a => a.actual).ToList()
        };
    });
}
