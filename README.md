# Risk Measures for Index Tranches and Bespoke CDOs


The purpose of the model is to calculate the credit spread sensitivity, correlation sensitivity, and default sensitivity via analytic methods for index CDO trades and bespoke CDO trades. 

The credit spread sensitivity is defined as the change in the MTM by perturbing the credit spread by a small amount; the default sensitivity is calculated by assuming that one obligor in the collateral pool defaults right away; and the correlation sensitivity is computed by perturbing the index base correlations by a small amount. 

The submitted new model is a great improvement of the existing risk measures in several aspects:

•	The new components developed in the Oscar/Fritz credit library enable us to capture risk more accurately. Previously, the index basis adjustment, base correlation calibration, and CDO2&3 trade equivalent CDO re-flattening (RE-CDO) were implemented outside the Oscar/Fritz library and there was no way to make corresponding adjustments in perturbed scenarios. In the new model, we can recalculate the basis adjustments, remap base correlations, and re-flatten RE-CDOs in the perturbed scenario. 

•	The credit spread sensitivity is switched to a bucketed one (CSPDH). In the model, parallel shift credit spread sensitivity has been used for many years. However, recent development in the market, especially the popularity of longer term trades, makes this measure inaccurate.
 
•	A new method to compute credit spread and correlation sensitivities are adopted in the submitted model. As outlined in the Section 1, those sensitivities are calculated by separating all relevant risk factors in the form of Jacobians and then combining them together, instead of a full bump and revaluation approach.

The advantage of the analytic sensitivity model is the computational efficiency, which allows us to compute the sensitivities by taking into account of all relevant risk factors. For example, it would be too time consuming to do a full bump and revaluation approach for a CDO2&3 trade. 

In the analytic sensitivity model, the internal shock amount is set to be  on a term node. Considering the fact that a CDO may have more than a hundred obligors, this shock amount is so tiny that, from the viewpoint of mathematical modeling, it is a “purer” delta than what we normally think of. When we use this more or less “theoretical” limit delta for any practical purpose, where a larger finite change in credit spread is encountered, we should pay more attention to any possible gamma effect. Normally this can be checked by a full bump/revaluation approach. 

Note that it is the first time that we can generate bucketed credit spread sensitivities for a structured trade. Aggregating CSPDH may lead to some no-arbitrage free situation. For example, now we have bucketed credit spread sensitivities and aggregate them for a changed credit spread curve. However, it does not contain any information if the changed curve is arbitrage free or not. Normally it is not a concern in the case of computing sensitivities with real market data and applies them to real market data, where the arbitrage free curves are ensured. 

Define a collateral pool of a CDO trade as a set of N reference names, , in which each reference name is described by a credit spread curve  , a recovery rate  , a default time  , and a notional amount  .  A CDO trade with this collateral pool as the underlying collateral pool has an attachment point   and a detachment point , and a fixed time horizon   with fee payments at interval .  Note that for the purpose of simplicity, this trade may refer to an index CDO trade, a bespoke trade, and a risk equivalent CDO (RE-CDO) for a CDO2&3 trade.

The base correlation approach is the market standard way to value a CDO trade. We assume the base correlations for the attachment point and the detachment point are  and , respectively. For an index trade, it can be directly calculated by an interpolation/extrapolation of the index base correlation. For a bespoke CDO trade and an RE-CDO for a CDO2&3 trade, a mapping methodology has to be employed. 

Within the current credit derivatives framework a reduced form of default probability of a reference name has been implemented and well maintained [4].  For the   reference name in the collateral pool, the hazard rate curve  is defined such that the default probability between s and s+ds  . With this definition the default probability functions built upon the hazard rate are

(1)	 = Survival probability,
(2)	 = Default probability,
(3)	  = Default probability density function.

For each credit spread curve, there is a standard term structure defined as  ={1w, 1m, 3m, 6m, 1y, 2y, 3y, 4y, 5y, 7y, 10y, 20y}).  The hazard rate is assumed to a stepwise linear function. For the   obligor, its   term is defined as  .

The normal copula semi-closed form model is the standard model in Oscar/Fritz to value a CDO trade. By employing a recursion algorithm, the expected loss of a collateral pool over time is built conditional on a latent variable. The detailed information of this model can be found in Refs. [6-7].

Assume a base tranche  with base correlation . Conditional on the latent variable Z, the expected loss at time s can be expressed as . The computation of this value at any time t can be found in Refs. [6-7]. Regardless of the instrument, the value of protection and Val01 can be expressed as a function of a series of , denoted as

(4)	 

(5)	 

In the model,   is a time series with a quarterly increment to the trade maturity.  Taking the value of protection as an example, we show how the analytic sensitivity works as follows. 

Assume the hazard rate curve of the   node of the ith obligor is perturbed by a small amount . The new hazard rate curve is denoted, , which can be expressed as  at node  .

Conditional on the latent variable, the change of the conditional default probability of the obligor can be written as:

(6)	 .


Using the recursion algorithm, we can work out the conditional expected base tranche loss over time when the hazard rate curve is perturbed. The change can be expressed as

(7)	 

For the same reason, we can write the change of the base tranche value, incurred by the change of this hazard rate, as follow:

(8)	 

The unconditional value of protection can be written as 

(9)	 

Note that the same set of the equations for the fee leg value Val01 can be deducted. For each obligor, we calculate all the sensitivities for each term node. 

Note that if the trade is an index trade, we have   and . For a bespoke trade, we have . However, , because the base correlation is mapped from the index base correlations. In the mapping process, the expected losses of both the collateral pool and the base tranche have to be used. When the credit spread curve is perturbed, they will change, leading to a change to the mapped base correlations. 

For a RE-CDO of a CDO2&3 trade,  . The attachment point of the RE-CDO, which we can value using the above discussed method, will change if we change the credit spread of an underlying obligor. 

General comments on the methodology:

1.	The new method allows us to directly perturb the condition default probability in the recursion. This will make the model much more computationally efficient than a regular bump and revaluation approach. 

2.	In the model, we have many layers of Jacobians built to calculate analytic sensitivities. Although mathematically speaking we can do as many as we want, practically it will incur two uncertainties. First, although many Jacobians can be calculated analytically, some of them are calculated numerically by assuming a finite shock to the relevant factors. In the model, the numerical shock is set to be . This treatment has to be based on the assumption that all the sensitivities are linear with respect to the relevant factors.  As we found, this is a good approximation. The second uncertainty is the numerical resolution. The computation of each Jacobian has certain numerical resolutions or error tolerances. Putting all the Jacobians together will lead to an aggregation of error tolerance, requiring a better computational error control than a regular bump and revaluation approach (reference: https://finpricing.com/lib/FiZeroBond.html).

3.	The model separates all the intermediate variables in the form of Jacobians, by assuming a linear relationship. Firstly this is an approximation and, secondly, the ignored high order joint effect of these intermediate variables might be of the same order as regular deltas.  An example is the credit spread sensitivity for a bespoke trade. A credit spread shock has two effects: remapped base correlations and expected losses. In the model, separating base correlation sensitivity and trade expected loss sensitivity with respect to the credit spread ignores the joint effect of the change in the mapped correlation and expected loss, which might be large in certain circumstances. 

4.	Aggregating all these sensitivities may lead to the violation of arbitrage free condition. For example, now we have bucketed credit spread sensitivities, and aggregate them for a changed credit spread curve. However, it does not contain any information about whether the changed curve is arbitrage free or not. Another example is that, as we know, a joint movement of credit spread and correlation can also create unrealistic movements. Therefore we should be cautious about the use of these sensitivities. Normally it is not a concern when computing sensitivity with real market data and applying them to real market data, where the arbitrage free curves are guaranteed by the market. 

It is important to note that, although there are uncertainties associated with the model, there is always a benchmark measure, i.e. a full bump and revaluation approach. When the shocks are larger, it is quite likely that the Gamma effect will kick in and the risk measures calculated using the method will no longer be accurate.  Therefore, anyone who uses the sensitivities should bear in mind that they are just deltas, which might be off in the case of large perturbation.

The credit spread sensitivity is defined as the change of MTM when the credit spread curve of an obligor is shocked by a small amount. Assume the credit spread curve of the   node of the ith obligor is perturbed by a small amount , the credit spread sensitivity can be expressed as

(10)	 
The computation of the present value for different type of trade is different and outlined in the following section. 

Suppose we have calculated the sensitivity of base tranche value of protection and Val01 to the change of all hazard rate nodes. The sensitivities to the credit spread curves can then be expressed as

(11)	 

The Jacobian  can be calculated using the single name CDS valuation model and Jacobians of base tranche values of protection and Val01 for two base tranches (with detachments D and A, respectively) can be calculated via Eqs.(6)-(9).

There is a basis adjustment in the computation of the index trade. The adjusted single name hazard rate curves can be then used to calculate the base correlation curves and MTM of the index trade. 

For an index trade, we assume that the adjusted hazard rate curves, which is described by   for the ith obligor at time s, has been calibrated.  By employing these adjusted curves, we can calculate the Jacobians of the tranche value with respect to the adjusted curves. In order to calculate the credit spread sensitivity for the obligor, we need to calculate the Jacobian against the original hazard rate curve. For example, for value of protection we need to calculate 

(12)	 ,

where   is the sensitivity directly calculated using CDO valuation model. We need to find out  . 

A term structure of scaling factor to the hazard rate curve is employed. It is defined as  with  (=3y, 5y, 7y, and 10y) being the index maturities. Between   and  , we have the following relationship 

(13)	 .

Therefore we can write

(14)	 

The calculation of   is dependent upon the same assumption of how to manage the basis risk. There are several choices:

•	Constant Adjustment Factor

We take  , which implies that

(15)	 .

With this choice, we make a corresponding change to the index level when the credit spread curve is adjusted by holding scaling factor unchanged.

•	Constant Index Basis

We assume the index basis is constant. The index basis is defined as the difference between the theoretical index spread, which is calculated directly using unadjusted curves, and the market quoted index spread. Suppose the theoretical index level be and the market quoted spread  , the index basis is defined as

(16)	 

Then in a perturbed scenario with shocked credit spread of an obligor, the corresponding changed new index spread becomes , with   being adjusted because of the changed constituent curves. 

In the model, two approximations are used to calculate Eq. (14). The first one is that the derivative of the new index level with respect to the constituent hazard rate curves is assumed to be constant.  The other is that the derivatives of the constituent single name CDS value with respect to the adjusted and unadjusted curves, respectively, are the same. Armed with these two approximations,   can be calculated via the following equation

(17)	 

  is the index MTM calculated using the adjusted constituent curves, which are defined as   an array of all adjusted hazard rate curves.  is the weight of the ith constituent obligor in the index collateral pool.  Note that a detailed description of how to derive this equation can be found in Ref. [2]. 

The submitted model goes with Constant Index Basis. As we will show in Section 2, the difference between this method and constant scaling factor method is small and the approximations of holding a constant derivative for index value and constituent single obligors are acceptable. 

For a bespoke trade, a change in the credit spread will have two effects. First, the expected loss has to be changed. Also, the mapped base correlations have to be adjusted because the base correlation mapping methodology is dependent on the expected losses of both entire portfolio and target base tranches. Previously approved credit spread sensitivity assumes no change of the base correlations in the perturbed scenario, because at that time the infrastructure did not allow us to remap the base correlations [5]. It is a great improvement to re-map the base correlations.

In the model, the influence of the remapped base correlations in the credit spread sensitivity can be calculated by

(18)	 ,

 for each perturbed hazard rate curve scenario and they are aggregated through Eqs. (8) – (11). Note that, in order to value a CDO trade, similar equations are need to calculated Val01 for two base tranches.


The correlation sensitivity is defined as the change of MTM when the base correlations are shocked by a small amount.  For an index CDO trade, the methodology remains unchanged. The correlation sensitivities are calculated by perturbing the two base tranche correlations by a small amount.

For a bespoke trade, two steps are involved in the correlation sensitivity computation. First, the sensitivities with respect to a small change in the mapped base correlations like that for an on-the-run index trade. For value of protection of a base tranche it is denoted as

(19)	 .

In the second step, this correlation sensitivity is mapped to the index base correlations. The base correlation mapping method can be viewed as a function linking index base correlations  to the mapped correlation.    is defined as a vector in the spaces with index ={CDX, iTraxx},  , and  for CDX and  for iTraxx. We can write this function as

(20)		 .

Therefore, a Jacobian can be worked out as

(21)	 

Once we calculate the correlation sensitivity with respect to the mapped correlations, the correlation sensitivity with respect to the index base correlations can be worked out straightforwardly by transposing the above matrix.

The default sensitivity is a kind of stress tests matching situations in which a credit default event has occurred or is perceived to be imminent. Within the current market standard modeling framework, a loss given default (  for the ith obligor) is claimed in the event of default. The principal of most junior tranche is reduced by   and the ith obligor is excluded from the collateral pool. 

By definition, the default sensitivity for the ith obligor can be expressed as

(22)	 .
 
When calculating  , the target trade has a new attachment point   and a new detachment point . MTM is calculated within the standard base correlation framework.

Compared with the initial version of default sensitivity [5], there are two major changes in the methodology:

1.	The base correlations are remapped in the perturbed scenario. Within the base correlation modeling framework, basically there are two effects we should consider in the event of default. Not only the tranche value has to be adjusted, the mapped base correlations have to be adjusted as well. Because one LGD can incur a large change in the expected losses, the default sensitivity with and without correlation adjustment are materially different and should be considered as two different risk measures. 

2.	The way to simulate a default is changed such that an instant default is achieved. Initially the default curve is modeled by shocking the hazard rate curve to a very high level. The change will have a small impact on the equity tranche.


