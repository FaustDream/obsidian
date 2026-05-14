
合同清单 查父项数据
 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识
  FROM icci_contractlist  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
  and (b.isLastLevel=false or b.isLastLevel is null or  b.isLastLevel='' )
    AND a.joinBudgetaryEstimates is not null

  ORDER BY d.abbreviation, c.contractNumber

报量清单 查父项数据

 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识   FROM icci_settlementstandingbook  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
  and (b.isLastLevel=false or b.isLastLevel is null or  b.isLastLevel='' )
      AND a.joinBudgetaryEstimates is not null

  ORDER BY d.abbreviation, c.contractNumber
  
  付款清单 查父项数据

 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识  FROM icci_paymentsliplist  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
  and (b.isLastLevel=false or b.isLastLevel is null or  b.isLastLevel='' )
      AND a.joinBudgetaryEstimates is not null

  ORDER BY d.abbreviation 

合同清单 查挂的概算挂错项目概算的数据


 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识  FROM icci_contractlist  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
   and b.joinProjectEngineering !=a.joinProjectEngineering 
       AND a.joinBudgetaryEstimates is not null

  ORDER BY d.abbreviation, c.contractNumber
  
  报量清单 查挂的概算挂错项目概算的数据

  
 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识   FROM icci_settlementstandingbook  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
    and b.joinProjectEngineering !=a.joinProjectEngineering 
	    AND a.joinBudgetaryEstimates is not null

  ORDER BY d.abbreviation, c.contractNumber
    付款清单 查挂的概算挂错项目概算的数据

  
 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识  FROM icci_paymentsliplist  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
    and b.joinProjectEngineering !=a.joinProjectEngineering 
	    AND a.joinBudgetaryEstimates is not null

  ORDER BY d.abbreviation, c.contractNumber
    报量清单 查概算版本挂错了的数据

 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识  FROM icci_settlementstandingbook  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
   and b.versions!='2024新版概算'
       AND a.joinBudgetaryEstimates is not null

   ORDER BY d.abbreviation 

      合同清单 查概算版本挂错了的数据
	  
 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识   FROM icci_contractlist a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
   and b.versions!='2024新版概算'
       AND a.joinBudgetaryEstimates is not null

   ORDER BY d.abbreviation, c.contractNumber
         付款清单 查概算版本挂错了的数据
	  
 SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识   FROM icci_paymentsliplist a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
   and b.versions!='2024新版概算'
       AND a.joinBudgetaryEstimates is not null

   ORDER BY d.abbreviation, c.contractNumber
   
   
   合同清单 父级挂接了概算
    SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识
  FROM icci_contractlist  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
  'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'
)
  and a.childFlag=false    and (a.joinBudgetaryEstimates is not null or a.joinBudgetaryEstimates!='') 
      AND a.joinBudgetaryEstimates is not null

  ORDER BY d.abbreviation, c.contractNumber
  
  
    合同清单 作废的数据
    SELECT d.abbreviation as 项目,c.contractNumber as 合同编号,c.serviceContractName as 合同名称,b.estimatesName as 概算名称,a.name as 清单数据标题,b.name as 概算数据标题,a.modifiedTime as 修改时间,CASE WHEN b.isLastLevel = 1 THEN '是' ELSE '否' END AS 概算子项标识
  FROM icci_contractlist  a left join icci_budgetaryestimates b on b.id=a.joinBudgetaryEstimates    left join icci_engineeringcontrac c on a.joinEngineeringContract=c.id left join ipim_projectengineerintree d on a.joinProjectEngineering=d.id
  WHERE a.state='有效' and a.joinProjectEngineering IN (
'e6678096571d455caebf2b772fc42f21',
  '9531223ae0d1483992ee84ac4a3cf5dc',
  '56865ad3df1b4a088ae00667dd75a893',
  '85f17506adee44c796d45fab20c566f4',
  '178a6c94ae314fefad6a9a2832e85462',
  '0052b59992404c8e8601a2ca128304ca',
  '30748aab604e489da04305b606e55ff6',
  '4a5fa3f1fa5e47499f213733e98ea3dc',
  'cbc432155b60459c90b6c49460989968',
  'cf3f4008850e4b03adfe3f01bf8fb8a8',
  'd7d2211b7588413c93e87b52a26935b7',
  'ed0842c625e24475b1c341d863f435cd',
  'ed26e98bb3a1439db5d21f39ec549d96',
  '6aa10488f1854540a07e6423f84b20d8',
  '4e6f9b4cce93472bb88492fa9dbf67c6',
  'c8e51716d9134a309314b331553df251',
  '819705681329428d8a0d2449f944e246',
  '57ef85ee788b4e8ba316d6e8c23c8161'  
)
  and a.childFlag=false    and (a.joinBudgetaryEstimates is not null or a.joinBudgetaryEstimates!='') and a.sequenceStatus ='CANCELLED'
  
  ORDER BY d.abbreviation, c.contractNumber
  
  
  
  -- 返回判断变价审批的与合同清单数据变更减少对应不上的

  
  SELECT 
    t1.contractNumber,
    t1.serviceContractName,
    t1.reduceAmount,
    t2.projTZJR1
FROM 
    (
        SELECT 
            b.id AS joinEngineeringContract,
            b.contractNumber,
            b.serviceContractName,
            SUM(a.reduceAmount) AS reduceAmount
        FROM 
            icci_pricechangeapproval a 
            LEFT JOIN icci_engineeringcontrac b ON a.joinEngineeringContract = b.id 
        WHERE 
            a.state = '有效'  
        GROUP BY 
            b.id, b.contractNumber, b.serviceContractName
    ) t1
JOIN 
    (
        SELECT 
            c.id AS joinEngineeringContract,
            c.contractNumber,
            c.serviceContractName,
            COALESCE(SUM(b.projTZJR1), 0) AS projTZJR1
        FROM 
            icci_contractlist a  
            JOIN icci_contractlistdetailed b ON b.parentId = a.id 
            LEFT JOIN icci_engineeringcontrac c ON a.joinEngineeringContract = c.id  
        WHERE  
            a.state = '有效'  
        GROUP BY 
            c.id, c.contractNumber, c.serviceContractName
    ) t2 
ON 
    t1.joinEngineeringContract = t2.joinEngineeringContract
WHERE 
    COALESCE(t1.reduceAmount, 0) != COALESCE(t2.projTZJR1, 0)