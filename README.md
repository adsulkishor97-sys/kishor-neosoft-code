 private static async Task<List<ConvertedKpiItemPlantDetails>> GenerateConvertedMaintenancePlantReport(KpiFormula kpiFormula, KpiDetail report)
        {
Uncovered code
            return await Task.Run(() =>
            {
                var converted = new List<ConvertedKpiItemPlantDetails>();
                foreach (var item in report.plants ?? new List<Plant>())
                {
                    var yValue = kpiFormula.formula!(item.actual); // Convert % to decimal for formula
                    if (yValue < 0)
                        yValue = 0;
                    converted.Add(new ConvertedKpiItemPlantDetails
                    {
                        // kpi = item.kpi,
                        plant = item.plantName,
                        actualY = yValue,
                        target = "100%",
                        min = 0,
                        max = 2
                    });
                }
                return converted;
            });
        }
