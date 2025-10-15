// Corrected: use KpiFormula type
var kpiFormulas = fixture.Build<KpiFormula>()
    .With(x => x.name, "Energy Efficiency")
    .With(x => x.target, 100m)
    .CreateMany(2)
    .ToList();

service.Protected()
    .Setup<List<KpiFormula>>("GetKpiFormulas", ItExpr.IsAny<KpiFormulaTargetRequest>())
    .Returns(kpiFormulas);
