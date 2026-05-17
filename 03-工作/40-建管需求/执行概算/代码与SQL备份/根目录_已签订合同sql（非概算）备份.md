  SELECT
  a.id,
COALESCE(c.contractState, '执行中') AS contractStatus,
  a.contractNumber AS contractNumber,
  a.serviceContractName AS contractName,
  b.unitName AS contractorUnit,
  a.signingDate,
  COALESCE(a.oldContractSigningAmount,0) AS contractAmount,
  CASE
    WHEN c.contractState = '已结算' THEN COALESCE(d.paymentSdmoney,0) 
    ELSE  COALESCE(a.oldContractSigningAmount,0)
  END AS contractEstSettleAmount,
  COALESCE(d.paymentSdmoney, 0) AS contractTotalSettleAmount
FROM
  icci_engineeringcontrac a
   LEFT JOIN ipim_companyunit b ON a.vendor = b.id AND b.state = '有效' 
   LEFT JOIN icci_contractstate c ON a.joinContractState = c.id AND c.state = '有效'
   LEFT JOIN (
        SELECT 
          SUM(paymentSdmoney) AS paymentSdmoney, 
          joinEngineeringContract 
        FROM icci_paymentsliplist 
        WHERE state = '有效' AND joinProjectEngineering = :proid
        AND joinCostProj = '24f9ac5df7304f91952ba4653f452ff5'
        AND sequenceStatus in  ('PROCESSING','COMPLETED')
        GROUP BY joinEngineeringContract
    ) d ON d.joinEngineeringContract = a.id

WHERE
  a.joinProjectEngineering = :proid
  AND a.state = '有效' AND a.sequenceStatus in  ('PROCESSING','COMPLETED')
GROUP BY a.id 
ORDER BY a.signingDate DESC