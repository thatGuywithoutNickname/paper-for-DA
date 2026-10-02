# Thesis working draft

> Editorial status — 29 September 2026, first prose draft of Chapter 3, Research Goals. Chapters 1–3 now contain English prose drafts; the Chapter 2 revisions of 25 September remain in place. Chapter 3 defines the prediction task, four testable subgoals and the evidence needed to assess them. Its final specification and any numerical acceptance targets remain to be agreed with the supervisor; the intended one-to-two-page length still requires checking in the final thesis layout. Markdown is the working version for review; the author updates Word manually after confirmation. Chapter 2 was synchronized to Word on 25 September 2026, retaining the author’s Word edits outside the agreed replacement. The new Chapter 3 has not been synchronized to Word, Prism, LaTeX or the compiled PDF. Section 4.1.1 provides a worked Methods example with illustrative arithmetic. The evaluation principles have prose in 4.1.7, while exact case membership, metric definitions and experimental settings remain to be specified. Other Methods sections, Results, Discussion and Chapter 5 still contain writing notes. Notes marked **To develop** are planning instructions, not finished thesis prose.

> Agreed scope: rapid prediction of local accumulated equivalent plastic strain in SAC305 solder, with FEM references generated using an Anand material model. Geometry and loading protocol are fixed. The intended input domain includes temperature, excitation amplitude and PCB Young's modulus. Current fixed-modulus experiments are an intermediate stage; the broader modulus study requires the forthcoming FEM data. Fatigue-life prediction is not a required output of this thesis.

## Contents

1. [Introduction](#1-introduction)
2. [State of the art](#2-state-of-the-art)
   - 2.1 The preceding study: an established capability and the next questions
   - 2.2 From direct prediction to a structured strain field
     - [2.2.5 Learning rapidly varying condition–response relationships](#225-learning-rapidly-varying-conditionresponse-relationships)
   - 2.3 Learning from intermediate physical responses
   - 2.4 Learning the error left by the first prediction
   - 2.5 Commercial field surrogates and the use of physical information
   - 2.6 Synthesis: the questions carried into this thesis
3. [Research Goals](#3-research-goals)
   - [3.1 Starting point and delimitation](#31-starting-point-and-delimitation)
   - [3.2 Main research question and subgoals](#32-main-research-question-and-subgoals)
   - [3.3 Required outcomes and evaluation criteria](#33-required-outcomes-and-evaluation-criteria)
   - [3.4 Dependencies and approach](#34-dependencies-and-approach)
4. [Main Chapter of the Thesis](#4-main-chapter-of-the-thesis)
   - 4.1 Methods
   - 4.2 Experiments and Results
   - 4.3 Discussion
5. [Summary and Outlook](#5-summary-and-outlook)
6. [References](#6-references)

# 1. Introduction

Consider an engineer comparing two vibration amplitudes for the same electronic assembly. The solder connections, the assembly geometry and the form of the loading remain the same. What the engineer wants to see is how the deformation in the solder changes: where the largest response occurs, how concentrated it is, and how the rest of the solder region responds. A single average cannot answer all of these questions. The useful result is a map that assigns a response value to each location of interest.

A finite-element simulation can provide such a map for a specified operating condition. Repeating that calculation for many conditions, however, creates a practical motivation for a faster approximation. Building on the group's neural solder-strain study by Meier et al., this thesis investigates learning that approximation from a limited collection of completed simulations. [[1]](<https://doi.org/10.1109/EPTC67330.2025.11392225>) The same solder region and its response map will serve as a running example: first to define the prediction task, then to explain the model, and finally to show what evidence is needed before trusting its predictions.

## 1.1 Engineering background and motivation

Solder forms connections within an electronic assembly and carries mechanical deformation as the assembly is loaded. The present study concerns SAC305 solder subjected to a prescribed vibration-loading protocol. Its response is described numerically using an Anand material model. Temperature, excitation amplitude and the Young's modulus of the printed circuit board (PCB) define the intended family of operating conditions.

The quantity to be predicted is accumulated equivalent plastic strain. Strain describes deformation relative to the original dimensions. The accumulated equivalent plastic strain condenses the history of plastic deformation at a material point into a scalar measure. Evaluating this quantity at multiple locations reveals how unevenly plastic deformation is distributed through the observed region. In this thesis, the intended output is that spatial distribution at a specified stage of the loading protocol.

The finite-element method (FEM) obtains a numerical response by dividing a structure into small elements and solving the mechanical problem using its geometry, material laws, loads and constraints. The Anand model supplies the description of material response within that calculation; it is not itself a direct predictor of the complete assembly's strain map. The distinction matters because the machine learning model studied here approximates the resulting field for a family of operating conditions, rather than replacing the constitutive update inside the FEM solver. The specific FEM formulation and material parameters will be documented in Section 4.1.2. [[2]](<https://doi.org/10.1016/0749-6419(85)90004-X>)

For the running example, imagine changing the vibration amplitude while keeping temperature and PCB stiffness fixed. The desired comparison consists of two strain maps on the same set of locations. Additional temperature and stiffness combinations extend this question across the intended parameter domain. If a learned model can provide sufficiently accurate maps at conditions that were not used to fit it, it could support repeated response comparisons without running the full FEM calculation for each query. Establishing that computational benefit requires measuring both the preparation cost and the subsequent prediction cost.

## 1.2 Research problem and significance

Each completed simulation supplies one example of the relationship between an operating condition and a response field. A surrogate is an approximation fitted to such examples. Its central challenge is to predict a field at a new condition using relationships learned from the available ones. Reproducing the examples already seen during fitting is necessary but insufficient: the engineering purpose concerns conditions for which the answer is not yet supplied.

Spatial prediction makes this challenge more demanding than matching one summary value. A predicted map might have approximately the right overall response level while placing too much strain in one region and too little in another. Likewise, a model that is accurate at several amplitudes can still be inaccurate between them. The number and placement of the FEM cases, the representation of the field, and the way errors are assessed all influence whether the approximation is useful.

The present thesis continues the group's study by Meier et al. on neural prediction of solder-joint strain values and patterns under vibration. That work supplies a direct starting point for investigating how the field predictor can be developed further. Section 2.1 reviews its findings and open questions; the present contribution is framed around the effects of physical auxiliary supervision and staged correction on the complete prediction. [[1]](<https://doi.org/10.1109/EPTC67330.2025.11392225>)

The first idea investigated in this thesis is to make additional use of information that the simulations already provide. Alongside the local field, the model can be trained to predict physically meaningful intermediate responses, including an amplitude-transmission descriptor and a global plastic-response descriptor. These predictions can then inform the field predictor. The motivation is that learning related responses may help organise the relationship between external conditions and local deformation. At a new condition, however, the intermediate responses must themselves be predicted; their FEM reference values are not available as inputs. Their physical meaning therefore motivates a hypothesis about learning, rather than guaranteeing improved field accuracy.

The second idea is to learn a correction to the initial field prediction. An initial model may capture some response patterns while leaving errors that also vary systematically with the operating condition. A further model can attempt to predict those errors. This introduces another testable question: whether the remaining error is learnable at unseen conditions, and how training the initial predictor and the correction in stages affects their combined result.

The central research question is therefore: **how do physical auxiliary supervision and staged residual modelling affect the accuracy of local accumulated-plastic-strain predictions at unseen operating conditions within a FEM-supported domain, when the available simulation data are limited?** The contribution depends on distinguishing useful design choices from plausible choices that do not improve the intended output. A more accurate intermediate response, a lower training error or a more elaborate architecture is not by itself an answer to that question.

## 1.3 Scope and thesis organisation

The geometry, material-model formulation and loading protocol define the scope of the approximation. The intended input domain spans temperature, excitation amplitude and PCB Young's modulus. The current development experiments use a fixed modulus of 20 GPa; they can inform comparisons within that slice but cannot establish generalisation in the stiffness direction. The broader study requires additional FEM data and evaluation. Transfer to different geometries, arbitrary loading histories and operating conditions beyond the supported domain constitutes further prediction problems.

Two kinds of evidence must also remain distinct. Agreement with withheld FEM results assesses how faithfully the surrogate approximates the selected numerical model. Establishing how well that numerical model represents the physical assembly requires its own material, numerical and experimental validation. The thesis does not infer physical validity, fatigue life or a measured speedup solely from surrogate field accuracy.

Chapter 2 develops the ideas needed to understand this investigation, following the decisions involved in predicting the running example's strain map. Chapter 3 turns the remaining uncertainties into research goals and evaluation requirements. Chapter 4 explains the prediction procedure, presents controlled experiments and discusses what their results support. Chapter 5 returns to the engineering question, summarising the findings and identifying the limits and next steps justified by the evidence.

# 2. State of the art

Return to the engineer requesting a strain map at a new combination of temperature, vibration amplitude and PCB Young's modulus. The group's preceding study by Meier et al. already demonstrated that neural networks can learn solder-joint plastic strain patterns from FEM data. [[1]](<https://doi.org/10.1109/EPTC67330.2025.11392225>) The question carried forward is how to use a limited collection of completed simulations to predict that new map more accurately, including its local concentrations of high strain.

Section 2.1 establishes what the preceding study achieved and where difficulties remain. From that starting point, Section 2.2 asks how a model can learn a whole map from examples. Because the maps describe the same solder region, they may share spatial patterns whose contributions change with the operating condition. This leads to a second question: how can the model learn those changes, including intervals where a small change in the input may produce a large change in strain?

The FEM calculations provide more than the final strain map. They also describe how vibration is transmitted through the assembly and how much plastic deformation develops overall. Section 2.3 asks whether learning these intermediate responses can help the model predict the local field, and what difficulties arise when those responses must themselves be estimated at a new condition.

Even after an initial map has been predicted, some regions may still need an increase or a decrease. Section 2.4 considers whether a further model can learn these remaining errors. It then asks how the two models should be trained: if the initial prediction keeps changing, the correction it needs changes too.

Finally, Section 2.5 places these choices in the context of commercial field surrogates. It examines how learning from simulation outputs, using additional physical response targets and imposing governing equations provide different ways for physical information to enter learning. This comparison clarifies the role of the proposed auxiliary supervision. Section 2.6 brings the discussion together to identify what existing work helps us understand and what this thesis still needs to test.

## 2.1 The preceding study: an established capability and the next questions

### 2.1.1 What the group has already demonstrated

Meier et al. studied a flip-chip corner solder joint under vibration using FEM-generated training data. They developed separate networks for two aspects of the response: a recurrent neural network for characteristic plastic strain and a feedforward neural network for its spatial pattern. The spatial predictor used temperature, vibration amplitude and PCB Young's modulus to produce 48 cross-section sub-area averages. Its predictions were compared with FEM patterns and, in an illustrative comparison at similar loading conditions, observed vibration-test damage. [[1]](<https://doi.org/10.1109/EPTC67330.2025.11392225>)

The important starting capability is therefore already present: learning a spatial solder response from simulation examples. The present thesis builds on that result by investigating how the prediction is organised. Its contribution depends on what the proposed changes achieve under a common evaluation.

### 2.1.2 What remains to be improved

The preceding paper reports spatially uneven errors and occasional negative accumulated-equivalent-strain predictions, which were replaced by zero. Its discussion identifies opportunities concerning spatial extraction, admissibility and combined characteristic-value/pattern prediction; it also identifies physics-informed modelling as future work. [[1]](<https://doi.org/10.1109/EPTC67330.2025.11392225>)

These observations lead to a practical distinction. A map can reproduce the broad response while remaining inaccurate in particular regions. An overall response value can help describe the severity of deformation, yet leave its spatial distribution unresolved. Learning those two aspects together is consequently a plausible next step. Section 2.3 develops the present thesis's proposed organisation from the vibration-transmission problem and the distinction between overall response and local distribution; its value must be assessed through the resulting field.

The remaining local error suggests a further question for this thesis: after an initial prediction has been made, is there a systematic part of its error that another model can learn? The existence of error motivates examining a correction.

### 2.1.3 The prediction task carried forward

For the rest of this chapter, consider the same solder region at a new operating condition,

$$
\boldsymbol{\mu}=(T,E_{\mathrm{PCB}},A),
$$

where $T$ is temperature, $E_{\mathrm{PCB}}$ is PCB Young's modulus and $A$ is excitation amplitude. At a position $\boldsymbol{x}$, the required quantity is the local accumulated equivalent plastic strain $p(\boldsymbol{\mu},\boldsymbol{x})$ at the specified stage of the loading protocol. A field prediction assigns this quantity to all requested positions. Geometry and the form of the loading protocol remain fixed.

The current study uses 81 prescribed output locations. These values and the predecessor's 48 sub-area averages have different spatial definitions. Their counts and published errors cannot establish a performance ordering; a comparison of modelling approaches needs the same current targets and evaluation cases.

This defines the next question closely enough to examine it: given a limited collection of complete FEM fields, how should a model represent their dependence on operating conditions and position?

## 2.2 From direct prediction to a structured strain field

### 2.2.1 Learning a whole map from examples

Place the available FEM maps beside one another, each labelled with its operating condition. One complete map is one example of the relationship to be learned. Each simulated map serves as the FEM reference field for its condition: the target against which a prediction is compared. The model adjusts its internal coefficients using these examples; this fitting process is called training. Afterwards, prediction means evaluating the fitted relationship at a supplied condition.

The predecessor's feedforward approach provides a direct construction:

$$
\boldsymbol{\mu}\longmapsto
\left[\widehat{p}(\boldsymbol{\mu},\boldsymbol{x}_1),\ldots,
\widehat{p}(\boldsymbol{\mu},\boldsymbol{x}_M)\right].
$$

Here, $M$ is the number of output locations and the hat denotes a prediction. A feedforward network forms weighted combinations of its inputs, applies nonlinear transformations and repeats this process through successive layers. Training adjusts the weights to reduce disagreement with the corresponding FEM reference fields. The output values can share internal features, so direct prediction need not treat every location independently. [[3]](<https://www.deeplearningbook.org/>)

This is a meaningful comparator for the present fixed-location task.

### 2.2.2 The operating condition selects a map; position selects a value

Consider one operating condition and its predicted strain map. Moving from one location to another changes the value being queried, while the operating condition stays the same. Now keep the location fixed and change the temperature, amplitude or PCB modulus: the question becomes how the response at that location changes between maps. Operating conditions and spatial positions therefore have different roles in the prediction.

These roles can be made explicit by supplying both to a model,

$$
(\boldsymbol{\mu},\boldsymbol{x})
\longmapsto\widehat{p}(\boldsymbol{\mu},\boldsymbol{x}).
$$

Here, $\boldsymbol{\mu}=(T,E_{\mathrm{PCB}},A)$ is the operating-condition vector: $T$ is temperature, $E_{\mathrm{PCB}}$ is PCB Young's modulus, and $A$ is excitation amplitude. The vector $\boldsymbol{x}$ gives the spatial coordinates of the queried position in the solder region. The quantity $p$ is the local accumulated equivalent plastic strain at the specified stage of the loading protocol; the hat in $\widehat{p}$ identifies the model's estimate. Thus, $\widehat{p}(\boldsymbol{\mu},\boldsymbol{x})$ is one predicted strain value at the supplied condition and position. The arrow denotes the learned mapping from these inputs to that value.

Holding $\boldsymbol{\mu}$ fixed and changing $\boldsymbol{x}$ queries different positions on the same predicted map. Evaluating the mapping at all prescribed positions assembles the complete predicted field. Changing $\boldsymbol{\mu}$ requests a map for another operating condition.

A similar distinction is familiar from repeated FEM simulations of the same assembly. The geometry and mesh can be kept fixed while the loading or material parameters change. Each queried position still refers to the same place in the structure, but the computed response there can change with the condition.

For the learned predictor, this shared spatial setting suggests a modelling question: can the different strain maps be assembled from common spatial patterns, with the contribution of each pattern changing with the operating condition? Each pattern can be expressed as a function of position, and a coefficient controls how strongly it contributes to the map. The construction below learns both these patterns and the relationship between operating conditions and their coefficients from the available FEM maps.

### 2.2.3 DeepONet: combining condition-dependent coefficients with spatial functions

DeepONet implements this construction with two networks. In the parameter-conditioned form considered here, the branch receives the operating condition $\boldsymbol{\mu}$ and produces coefficients. The trunk receives the query position $\boldsymbol{x}$ and evaluates spatial functions there. Multiplying corresponding outputs and summing them gives the predicted value:

$$
\widehat{p}(\boldsymbol{\mu},\boldsymbol{x})
=b_0+\sum_{k=1}^{r}b_k(\boldsymbol{\mu})\,t_k(\boldsymbol{x}).
$$

Here, $b_k$ are the branch coefficients, $t_k$ are the trunk functions, $r$ is the number of paired outputs, and $b_0$ is a learned scalar bias. Lu et al. introduced this structure for learning operators, with sampled input functions supplied to the branch. The present form supplies operating parameters instead. [[4]](<https://doi.org/10.1038/s42256-021-00302-5>)

For a given condition, the branch coefficients are shared across all queried positions. As an illustrative calculation, suppose two trunk functions take the values $(1,1,1)$ and $(0,0,1)$ at three locations. The first contributes equally at all three locations; the second contributes only at the last. With branch coefficients 2 and 3 and zero bias, the map is

$$
2(1,1,1)+3(0,0,1)=(2,2,5).
$$

At another condition, the branch can produce different coefficients, changing the map. During training, disagreement with the FEM reference fields adjusts both networks, so the spatial functions and the condition-to-coefficient relationship are learned together.

### 2.2.4 What the related mechanics evidence supports

Koric et al. applied DeepONet to stress-field prediction in small-strain plastic deformation with varying loads and material properties. This supplies a relevant mechanics precedent for representing nonlinear fields through a branch–trunk model. The material formulation, stress target and sampling design differ from those of the present accumulated-strain task, so the result motivates investigation here without settling the comparison. [[5]](<https://doi.org/10.1007/s00366-023-01822-x>)

The present task concerns operating conditions and positions within a fixed geometry and loading protocol. The trunk's coordinates locate a query within that geometry; they do not describe a changing geometry. Accuracy at the 81 observed locations therefore leaves accuracy at untested positions or on another geometry to be evaluated separately.

The branch–trunk construction explains how a model can assemble a strain map from operating conditions and position. Its accuracy also depends on learning how the branch coefficients change between conditions. This raises a further question when the response varies rapidly over part of the operating domain.

### 2.2.5 Learning rapidly varying condition–response relationships

Return to the engineer changing the vibration amplitude. At a fixed temperature, PCB modulus and position in the solder, the strain may grow slowly over one amplitude interval and much faster over another. Changing the temperature may also move the interval of rapid growth. A field predictor must learn how the map changes through these intervals. In the DeepONet representation, this places a demand on the branch: its coefficients must change appropriately with the operating condition. 

A conventional multilayer perceptron (MLP) can represent a steep rise. With limited FEM examples and a given training procedure, however, it may reproduce the overall trend while predicting a sharp rise too gradually. Correcting that local shape while retaining the fit elsewhere may require coordinated changes to several neurons' parameters. This motivates exploring a representation with parameters that adjust parts of an internal response curve more directly.

To understand one way of making such an adjustment, begin with a single input and output. A spline joins small polynomial segments into a smooth curve. The points separating the segments are called knots. In a B-spline representation, coefficients control the contributions of overlapping basis functions, each active over a limited input interval. With the knots and other coefficients fixed, changing one coefficient changes only part of the curve. Figure 2.1 illustrates this local adjustment, which is part of the motivation for using splines within a network. [[6]](<https://proceedings.iclr.cc/paper_files/paper/2025/hash/afaed89642ea100935e39d39a4da602c-Abstract-Conference.html>)

![Two panels show the same steep continuous target and a spline approximation before and after a local coefficient adjustment. The local discrepancy decreases while the functions remain unchanged outside the marked support.](figures/kan-local-spline-adjustment-v2.png)

Figure 2.1. Local adjustment of a spline approximation. (a) The initial curve underestimates the target near its rapid rise. (b) Changing one cubic-spline coefficient reduces the discrepancy; the bracket marks the corresponding basis function's support. Shading shows the remaining discrepancy, and dots on the right-hand input axis mark knots. Both panels use the same coordinates and target. The adjustment is constructed for explanation, not obtained by training; these curves are neither FEM data nor an MLP–KAN performance comparison.

To see where such a curve can enter a network, compare two ways of processing incoming values. As shown in Figure 2.2(a), an MLP unit multiplies incoming values by learned weights, adds the contributions and a bias, then applies a nonlinear activation function. Training changes the weights and bias, while the activation's functional form, such as tanh, is specified in advance. [[3]](<https://www.deeplearningbook.org/>)

Liu et al.'s Kolmogorov–Arnold network (KAN) instead places a learnable one-dimensional function on each connection, also called an edge. In Figure 2.2(b), each incoming value passes through its own curve, and the resulting values are summed. The original implementation combines a spline with an additional base function. The enlarged spline in the figure repeats the adjusted curve from Figure 2.1, linking the local adjustment to its place in the network. [[6]](<https://proceedings.iclr.cc/paper_files/paper/2025/hash/afaed89642ea100935e39d39a4da602c-Abstract-Conference.html>)

![A minimal MLP unit uses learned scalar weights before summation and a fixed activation. A KAN unit applies learned curves on its incoming connections before summation. Enlargements compare a fixed activation shape with the spline from Figure 2.1.](figures/mlp-kan-function-placement-v2.png)

Figure 2.2. Where nonlinear functions occur in MLP and KAN units. Inputs are labelled x₁ and x₂, h denotes the output, and Σ denotes summation. In (a), w₁ and w₂ are learned scalar weights; the bias is omitted, and the inset illustrates a normalised tanh shape. In (b), φ₁ and φ₂ are learned scalar functions. The lower inset repeats Figure 2.1's adjusted spline and marks the same local support. It illustrates the spline component; an additional base component is omitted. The diagrams show computation units, not complete branch architectures or measured prediction results.

For the illustrated KAN unit, the computation is simply

$$
h=\phi_1(x_1)+\phi_2(x_2),
$$

where each $\phi_i$ transforms one scalar input $x_i$, and $h$ is their sum. Connecting successive layers lets later functions act on combinations produced by earlier ones. The local support in Figure 2.1 belongs to one spline input coordinate. It does not divide the physical temperature–amplitude domain into independent regions.

There is already a connection to operator learning. Abueidda et al. introduced DeepOKAN, using KANs in both the branch and trunk of a DeepONet. They used Gaussian radial basis functions and evaluated wave, elasticity and transient Poisson examples. This provides a precedent for combining learned edge functions with a branch–trunk representation. It does not establish the performance of a spline-based branch on the present plastic-strain task. [[7]](<https://doi.org/10.1016/j.cma.2024.117699>)

For this thesis, a spline-based KAN branch provides a way to test whether these locally adjustable curves help learn the condition-dependent coefficients from limited FEM examples. Chapter 4 specifies the comparison with an MLP branch, including spline grids and training procedures, and evaluates the learning behaviour and the accuracy of the complete field at unseen operating conditions. The following section turns to another source of guidance: the intermediate physical responses already available in the FEM simulations.

## 2.3 Learning from intermediate physical responses

### 2.3.1 How physical information can guide learning

Consider again the strain map predicted for a completed FEM case. The model compares its prediction with the FEM result for the same operating condition. Training repeatedly adjusts the model parameters to reduce the difference. Using known reference values to guide learning in this way is called supervised learning. The prediction errors are combined into a numerical measure called a loss, which training seeks to reduce.

The preceding study proposes making further use of physical information. Governing equations provide an additional way to check predictions: the predicted motion, stress and deformation should satisfy the relevant mechanical relationships. Substituting network predictions into these equations measures how far they depart from those relationships. This departure is called an equation residual. In the typical physics-informed neural network (PINN) formulation of Raissi et al., equation residuals contribute to the loss, together with terms for the relevant initial conditions, boundary conditions and any supplied observations. Wang et al. apply this principle to DeepONet. [[8]](<https://doi.org/10.1016/j.jcp.2018.10.045>) [[9]](<https://doi.org/10.1126/sciadv.abi8605>)

Applying the full mechanical equations to the present task would require additional work in two respects. First, the model currently predicts only the accumulated-plastic-strain field at the final time. That map does not provide the displacement, stress and evolving material-state information needed to evaluate the full equations. The predicted variables and their description over time would therefore need to be extended.

Second, even if the required variables were predicted, incorporating the full mechanical model into PINN training would remain a substantial task, it is technically hard to build a series of governing differential equations, that can describe multiple complicated physical behavior in a complex system. The equations describing structural motion would need to be coupled with the solder's nonlinear, history-dependent material response, together with the relevant initial, boundary and interface conditions. These relationships would then have to be implemented and verified as constraints on the network predictions.

The existing FEM calculations already use mechanical equations and an Anand material model to compute the assembly's response. Alongside the final strain map, they provide information about how the structure vibrates and how the solder deforms plastically. These accompanying results offer another way to guide learning: selected responses can become additional quantities that the model learns to predict.

For example, a model could predict the vibration amplitude at an observed location and compare it with the amplitude obtained from FEM. The resulting error would contribute to training alongside the strain-map error. This thesis calls the use of such physical response targets physical auxiliary supervision. The strain map itself already comes from a physical simulation; the additional supervision makes selected response quantities explicit learning targets. It does not enforce the full mechanical equations, and whether it improves the final field remains an experimental question. The next section considers which responses could usefully guide strain-map prediction.

### 2.3.2 From vibration transmission to plastic response

Return to the vibrating assembly. An imposed excitation produces motion of the structure, and the resulting deformation loads the solder. The imposed amplitude alone therefore does not describe the motion near the solder region. This observation motivates first learning a quantity that characterises vibration transmission.

For the observed edge location, let $A_e$ denote the displacement-response amplitude and $A$ the imposed excitation amplitude, expressed in the same units. Their ratio $H_e$,

$$
H_e=\frac{A_e}{A},
$$

describes how strongly that location moves relative to the excitation. It supplies a compact measure of the observed vibration response.

A second question concerns the extent of plastic deformation: how much additional plastic strain accumulates, on average over the solder volume, during the final loading cycle? Denoting volume-averaged accumulated equivalent plastic strain by $\bar p_V(t)$, this quantity is

$$
g=\bar p_V(t_{\mathrm{end}})
-\bar p_V(t_{\mathrm{start}}),
$$

where the two times mark the end and start of the final cycle. This is a volume-averaged increment over one cycle, while the requested local field contains accumulated values at the final time. The 81 field-query locations do not define the volume average used here.

When predicted inside the model, $H_e$ and $g$ are called the transmission descriptor and the global plastic-response descriptor. These names denote the physical quantities estimated by the networks. Their response-extraction details belong to Chapter 4; the present purpose is to explain why learning them might help field prediction.

### 2.3.3 Organising prediction through intermediate responses

The two responses suggest a way to organise prediction. The model first estimates the edge-amplitude ratio from the operating condition. It then uses the condition and that predicted ratio to estimate the overall plastic-strain increment. Finally, the field predictor receives the operating condition and both intermediate predictions, together with the query position. Retaining the original condition allows the field prediction to use information beyond the two scalars.

This thesis calls that arrangement hierarchical because an intermediate prediction becomes information for a later part of the model. Its design motivation comes from the mechanical interpretation of vibration transmission and from using an overall response to guide a local prediction. The volume-averaged increment itself summarises local plastic evolution. Predicting it before the field is a modelling choice, rather than a claim that the average physically causes the local strains.

How are the intermediate predictions learned? For a completed FEM training case, the simulation provides both the strain map and the response quantities used to calculate $H_e$ and $g$. The network can therefore be checked against all three targets. At a new, unseen condition, only the operating inputs and fixed coordinates are supplied; the model must estimate the intermediate responses as well as the field.

| Stage | Role of the intermediate physical responses |
|---|---|
| Training on completed FEM cases | FEM values supervise the intermediate predictions; predicted values are passed to the field predictor. |
| Prediction at a new condition | The model estimates the intermediate values from the supplied inputs and uses those estimates to predict the field. |

For the field and the two auxiliary responses, the losses can be combined as

$$
\mathcal{L}
=\mathcal{L}_{\mathrm{field}}
+\lambda_H\mathcal{L}_{H}
+\lambda_g\mathcal{L}_{g}.
$$

Here, $\mathcal{L}$ is the total loss that training seeks to reduce. Its three components compare predictions with the corresponding FEM training references. The field loss $\mathcal{L}_{\mathrm{field}}$ measures disagreement in the local accumulated equivalent plastic strain field. The transmission loss $\mathcal{L}_{H}$ measures error in the predicted edge-amplitude ratio $H_e$; the subscript $H$ refers to that transmission quantity. The global plastic-response loss $\mathcal{L}_{g}$ measures error in the predicted quantity $g$, the volume-averaged accumulated-plastic-strain increment over the final loading cycle.

The nonnegative weights $\lambda_H$ and $\lambda_g$ multiply the transmission and global plastic-response losses, respectively. They set the contributions of those auxiliary errors relative to the field loss, whose coefficient is one in this expression. For fixed component-loss values, increasing a weight increases that term's contribution to the total; setting it to zero removes that term from the sum. Training therefore uses a weighted combination of the field and intermediate-response errors to adjust the model parameters. Chapter 4 specifies the target transformations, the exact loss definitions and the weight schedules used in each model.

Related research provides a comparison for using response information. He et al.'s material-response-informed DeepONet supplies single-crystal responses to help predict polycrystal response. That study supports investigating response information as part of a learned representation. Its supplied response inputs differ from the intermediate quantities predicted within the present hierarchy. [[10]](<https://doi.org/10.1007/s11837-024-06681-5>)

### 2.3.4 Limits of intermediate-response supervision

An overall response can help characterise a case while leaving its spatial distribution uncertain. For example, the illustrative fields $(1,3)$ and $(2,2)$ share the same arithmetic mean but have different maxima and distributions. This shows the spatial information lost through aggregation; the example does not redefine the volume-averaged cycle increment $g$. Local field supervision is still needed to learn where strain accumulates.

The hierarchy also passes prediction errors forward. If the estimated transmission or plastic response is inaccurate, the field predictor receives inaccurate intermediate information. Evaluation must therefore use the complete predicted path. Even when FEM response values exist for a withheld case, supplying them to the predictor would give it information unavailable in the intended application.

Learning several targets introduces a further interaction. Parameters involved in more than one prediction receive training signals from several losses, and an adjustment that helps one task can hinder another. GradNorm addresses differences in task-training behaviour by adjusting loss weights using gradients, which describe how parameter changes affect the losses. PCGrad addresses conflicting gradient directions. These studies identify concrete optimisation difficulties without establishing the best intervention for the present task. [[11]](<https://proceedings.mlr.press/v80/chen18a.html>) [[12]](<https://proceedings.neurips.cc/paper/2020/hash/3fe78a8acf5fda99de95303940a2420c-Abstract.html>)

The proposed hierarchy thus poses a testable question: does supervising and using these intermediate responses improve the final field at unseen conditions? Their physical meaning motivates the design; field evaluation determines its benefit. In particular, the accuracy of the largest local strains still needs to be checked. The next section starts from this concern and asks whether errors left by the initial field prediction can themselves be learned.

## 2.4 Learning the error left by the first prediction

### 2.4.1 Why a complete field prediction may still need correction

Return to the engineer inspecting the strain map. The accumulated equivalent plastic strain is unevenly distributed across the 81 observed positions, with local concentrations of high strain. Here, the reference hotspots are the positions with the three largest FEM strain values. Several may lie within the same region. Their values matter because they describe where plastic deformation accumulates most strongly.

A field predictor can capture the overall pattern while still missing part of a local peak. The starting point for residual learning is this remaining discrepancy: after predicting the complete field, can a second model learn what still needs to be added or subtracted? Figure 2.3 makes the question visible. The initial prediction retains the broad shape of the reference field, but underestimates its left peak and slightly overestimates its right side. Subtracting the prediction leaves a local positive correction on the left and a negative correction on the right.

![Three aligned plots show a reference field, an initial DeepONet prediction with the difference shaded, and their pointwise residual. All panels use the same vertical scale.](figures/field-to-residual.png)

Figure 2.3. From a complete field to the correction it requires. The dotted guide in (b) repeats the reference field; its shaded difference from the initial prediction becomes the residual in (c). The curves are a constructed one-dimensional illustration in the standardised logarithmic field coordinate, not FEM data or trained-model results. All panels share a vertical scale, allowing direct comparison of residual magnitudes and field values. The schematic position axis does not represent the ordering of the 81 observation points.

This example explains a reason to investigate correction, rather than establishing a limitation of MLPs or DeepONets. High strain and large prediction error are different properties: a hotspot may already be predicted accurately, and a lower-strain position may still need adjustment. The FEM reference, initial prediction and their difference must therefore be compared before deciding where correction is useful.

A second model also needs a relationship it can learn from the available inputs. For example, across nearby operating conditions, the initial model might repeatedly underestimate the same region, with the size of the error changing with vibration amplitude. Such a repeatable dependence on the operating condition is what would make the remaining error a possible prediction target. Whether this behaviour occurs in the present data, and whether learning it improves unseen-case predictions, are questions for the experiments.

### 2.4.2 Turning a remaining discrepancy into a second target

For a completed training case, the FEM field and the initial prediction are both available. Subtracting them at every observed position gives the residual. This subtraction takes place in the representation used for learning: the standardised logarithmic field, denoted by $z$. Write the operating inputs as $\boldsymbol{\mu}$, comprising temperature, excitation amplitude and PCB Young's modulus, and the spatial coordinate as $\mathbf{x}$. The reference residual is

$$
r(\boldsymbol{\mu},\mathbf{x})=z(\boldsymbol{\mu},\mathbf{x})-\hat z_b(\boldsymbol{\mu},\mathbf{x}).
$$

Here, $z(\boldsymbol{\mu},\mathbf{x})$ is the transformed FEM reference value, $\hat z_b(\boldsymbol{\mu},\mathbf{x})$ is the initial model prediction, and $r(\boldsymbol{\mu},\mathbf{x})$ is the correction that would remove their difference exactly. The subscript $b$ identifies the initial model, and a hat denotes a predicted field quantity. For a hierarchical predictor, the initial field uses its own intermediate-response predictions, so this residual includes the errors of the complete prediction path.

The corrector learns to estimate that residual. Denoting its output by $\hat r$, the combined prediction is

$$
\hat z(\boldsymbol{\mu},\mathbf{x})=\hat z_b(\boldsymbol{\mu},\mathbf{x})+\hat r(\boldsymbol{\mu},\mathbf{x}).
$$

Thus, $\hat r$ is the predicted correction and $\hat z$ is the corrected field, both in the same transformed coordinate as $z$. A positive correction raises the predicted strain after the inverse transformation; a negative correction lowers it. Chapter 4 specifies that transformation. Adding a correction in this coordinate is not the same operation as adding the same numerical value directly to physical strain.

During training, a completed FEM simulation supplies the reference field, so the required residual can be calculated and used as a target. The model learns how that correction depends on the operating inputs and any permitted features it predicts internally. At a new operating condition, it uses the learned relationship to estimate the correction. The new condition's true residual and FEM-derived hotspot locations are unavailable to this prediction process.

Once training is complete, the initial predictor and correction model together form a complete predictor. Given a new operating condition, they calculate the initial field and the estimated correction, add them in the transformed coordinate, and apply the inverse transformation to return the final strain field. This requires neither a new FEM reference nor another training run. The fitted network parameters and preprocessing remain fixed during prediction, while their outputs vary with the supplied condition.

There is a precedent for learning in this way. Under squared-error loss, Friedman's gradient-boosting formulation fits successive models to the residuals left by earlier predictions. [[13]](<https://doi.org/10.1214/aos/1013203451>) This establishes the connection to additive residual learning; the combination of DeepONet and RBF correction studied here has its own architecture and training schedule, whose benefit must be evaluated separately.

### 2.4.3 Giving the correction a spatial form with radial basis functions

The residual specifies what needs to change at each position. In Figure 2.3, the left region needs an increase, while the right region needs a decrease. To make either adjustment, consider a spatial shape whose influence is strongest near a chosen centre and gradually weakens with distance. A Gaussian radial basis function (RBF) supplies such a shape. One common form is

$$
\psi_k(\mathbf{x})=
\exp\!\left(-\frac{\|\mathbf{x}-\boldsymbol{\xi}_k\|^2}{2\sigma_k^2}\right).
$$

Here, $k$ identifies the spatial function, $\mathbf{x}$ is the query position, $\boldsymbol{\xi}_k$ is the centre of the function, and $\sigma_k>0$ controls its width. The norm $\|\mathbf{x}-\boldsymbol{\xi}_k\|$ measures the spatial distance from the query position to that centre in the solder region. At the centre, the distance is zero and the exponential equals one. Moving away makes the exponent more negative, so the function decreases towards zero. A larger width makes that decrease more gradual. Kernel approximation provides the mathematical background for combining such functions. [[14]](<https://doi.org/10.1017/S0962492906270016>)

This function supplies a positive spatial shape. To turn it into a correction, multiply it by a coefficient: a positive coefficient adds to the predicted field, a negative coefficient subtracts from it, and the coefficient's magnitude sets the strength of the contribution. The centre and width determine where that contribution acts most strongly and how far it extends. Several contributions can then be added to approximate the residual across the region.

How much each shape should contribute depends on the operating condition. A neural network predicts these coefficients from the condition and any permitted internally predicted features. With $r_c$ spatial functions, the estimated correction is

$$
\hat r(\boldsymbol{\mu},\mathbf{x})=\sum_{k=1}^{r_c}c_k(\boldsymbol{\mu})\,\psi_k(\mathbf{x}).
$$

Here, $r_c$ is the number of spatial functions and $c_k(\boldsymbol{\mu})$ is the predicted coefficient of the $k$th function for operating condition $\boldsymbol{\mu}$. The sum combines their signed contributions at the query position. Centres and widths are specified or fitted using permitted training information and held fixed when predicting an unseen condition. 

Figure 2.4 follows the same residual as Figure 2.3 and separates two questions. First, can the chosen spatial shapes represent it? In panel (a), a positive multiple of one Gaussian follows the left peak but cannot also supply the negative correction on the right. Changing its coefficient scales or reverses the whole contribution. Panel (b) adds a second Gaussian multiplied by a negative coefficient. The two contributions together reproduce this constructed residual.

![Four panels distinguish the capacity of one RBF, positive- and negative-coefficient contributions, errors in predicted coefficients, and the corrected field. The required residual is a continuous black line; exact reconstruction is marked with open circles.](figures/rbf-correction.png)

Figure 2.4. Representing and predicting a spatial correction. (a) The best least-squares coefficient for one fixed RBF still leaves the right-hand discrepancy. (b) A positive-coefficient contribution and a negative-coefficient contribution sum to the required residual; both Gaussian functions themselves are positive. (c) Open circles mark evaluations of that exact reconstruction, while the blue curve uses illustrative estimated coefficients. The required residual remains a continuous black line. (d) Adding the estimated correction improves this illustrative field but does not recover it exactly. Panels (a) to (c) enlarge the residual scale relative to Figure 2.3. All curves and coefficients are constructed examples, not results from training or FEM.

Second, can the network predict the right coefficients? Panel (c) shows why suitable spatial shapes are only part of the solution. The illustrative estimated coefficients have smaller magnitudes than those required, leaving part of the residual uncorrected. Panel (d) adds this estimated correction to the initial field. For completed FEM cases, representation can be examined separately by fitting the best coefficients to each known residual with the spatial functions fixed. That reconstruction diagnoses what the functions can express; prediction at a new condition requires estimating the coefficients without knowing its residual.

The one- and two-function examples make these roles visible with very few spatial shapes. The current full-node configuration extends the representation to cover all 81 output locations with 81 coefficients. Their contributions are assembled through a fixed, invertible 81-by-81 Full-Node Output-Coordinate Map. Any residual vector at those locations can therefore be represented by a suitable coefficient vector. This changes the coordinates in which the correction is learned without reducing the output dimension. The remaining question is whether the network can predict suitable coefficients at unseen conditions, as judged by the final field's hotspot and complete-field accuracy.

### 2.4.4 What changes when training is divided into stages

The correction target depends on what the initial predictor currently gets wrong. If that predictor continues learning, its residual can change too. Consider an illustrative training value of $z=1.0$ at one position. An initial prediction of $0.8$ requires a correction of $0.2$. If further training raises the initial prediction to $0.9$, the required correction becomes $0.1$. These invented values are in the transformed coordinate and show why the two learning tasks interact.

One way to give the correction model a stable target is to hold the initial predictor unchanged while learning the correction. The initial predictor is also called the base model. Its internal parameters, including weights and biases, are the numerical values adjusted during training to determine how inputs become predictions. Freezing the base means stopping updates to these parameters and retaining its fitted preprocessing, so that a given training case continues to produce the same initial field and residual.

This leads to a simple division into training stages: first fit the initial predictor, then freeze it and train the correction model using the residuals it leaves. Each stage has a clear task, and the second stage learns against a fixed reference discrepancy for each case.

A different staged procedure first fits the initial predictor and then allows selected parts of it to adjust together with the correction model. The residual can still be calculated from the current initial prediction, but now changes as that prediction changes. Training seeks a better combined field while both contributions adjust. 

Neither choice is automatically preferable. A frozen predictor may leave a difficult residual; partial joint adjustment may make the combined task easier, or may disturb useful features already learned. A model with a smaller initial error need not leave an easier error to correct. The relevant outcome is the complete prediction at unseen conditions.

This motivates comparisons between the initial model, the corrected model and the chosen training schedules. A suitable single-stage or capacity-matched comparator helps distinguish the effect of staging from the effect of adding adjustable parameters. The original concern remains the same throughout: does the complete predictor recover the local high strains more accurately while maintaining or improving the rest of the field?

## 2.5 Commercial field surrogates and the use of physical information

An engineer with a collection of completed simulations can use those results to train a model for predicting the response of a new design. Ansys SimAI provides a commercial example of this approach. Ansys describes it as physics-agnostic: the model learns from numerical-solver outputs, in contrast to methods that incorporate governing-equation penalties into training. Physical information therefore enters through the simulated examples. Learning these examples, however, does not itself enforce the governing equations on every new prediction. [[15]](<https://ansys.synopsys.com/blog/explaining-simai>)

From an engineering perspective, this approach offers a practical way to reuse existing simulation work. The numerical solver has already accounted for the material behaviour, loading and boundary conditions represented in each training case. A model can learn the resulting input–output relationship without those equations being reformulated as training constraints. This separation also offers a plausible route to broad reuse: a common learning workflow can accept simulation results from different physical domains and software tools. These properties fit SimAI's documented emphasis on archived data, support across physics domains and use without deep-learning expertise. [[16]](<https://ansyshelp.ansys.com/public/Views/Secured/SimAI/v000/en/SimAI_ug/SimAI_ug/C_UG_SAI_overview.html>)

Explicit equation constraints offer a different trade-off. They provide information beyond the available solution examples and can reduce the need for paired training data, as demonstrated for the PDEs studied with physics-informed DeepONets. [[9]](<https://doi.org/10.1126/sciadv.abi8605>) Their incorporation also introduces training challenges: different terms in a PINN loss can produce unbalanced gradients, making it difficult to satisfy the constraints together. [[17]](<https://doi.org/10.1137/20M1318043>) These findings help explain why explicit physical constraints are not an automatic improvement over learning from solver outputs. The appropriate choice depends on the available data, the equations that can be evaluated from the model outputs, and the difficulty of the resulting optimisation problem.

The data-driven route, in turn, depends on how well the examples cover the intended prediction task. Ansys's training guidance distinguishes broad design spaces requiring diverse data from narrowly trained models with limited performance outside their scope. [[18]](<https://ansyshelp.ansys.com/public/Views/Secured/SimAI/v000/en/SimAI_ug/SimAI_ug/C_UG_SAI_training_data_selection_best_practices.html>) For the present thesis, this brings the discussion back to a limited collection of FEM cases: can the intermediate physical responses already available in those cases make field prediction more effective? The proposed auxiliary supervision uses those responses as additional learning targets. Whether this improves the final strain field at unseen conditions remains the question to be tested.

## 2.6 Synthesis: the questions carried into this thesis

The opening question concerned the accuracy of a strain map at a new operating condition. The review has separated two parts of that problem. A model needs a way to express the spatial distribution, and it must learn how that distribution changes with temperature, amplitude and PCB modulus. The branch–trunk construction makes these roles explicit through shared spatial functions and condition-dependent coefficients. The spline discussion addresses how those coefficients might be learned when the response varies rapidly. A representation that can express a field still needs enough information and suitable training to predict it from the available examples.

The next question was whether the simulations offer useful guidance beyond the field itself. Vibration transmission and overall plastic response give physically meaningful quantities for the model to learn. They leave the local distribution partly undetermined, however, and their prediction errors can pass into the field predictor. Residual correction addresses another part of the problem: a discrepancy left by the initial prediction may have a repeatable dependence on the operating condition. Expressing a known discrepancy with spatial functions and predicting its coefficients at a new condition are separate tasks. Whether the initial predictor remains fixed or continues to adjust also changes the correction-learning problem.

These distinctions lead to the following questions for the present task.

| Question | What existing research contributes | What remains to be established for this task |
|---|---|---|
| How can the model represent a complete strain map? | Direct output prediction and the DeepONet branch–trunk construction | How accurately the selected representation captures the local field with limited FEM cases |
| How can it learn rapid changes between operating conditions? | Learnable spline functions and their use in operator networks | Whether a spline-based branch improves learning and unseen-condition field accuracy relative to an MLP branch |
| Can intermediate physical responses help predict the local field? | Response-informed representations and methods for learning several related targets | Whether the proposed hierarchy improves the field despite intermediate prediction errors and interactions between training objectives |
| Can a further model usefully correct the remaining error? | Additive residual learning and spatial RBF representations | Whether correction improves the complete prediction, and how freezing or adjusting the initial predictor during correction training affects that result |

The commercial-surrogate discussion places these questions in a wider context. Learning from FEM outputs already draws on simulations in which the mechanical problem has been solved. Auxiliary response targets make selected parts of that response explicit in training, while governing-equation constraints supply another form of guidance. This distinction explains the choice investigated here: use the available transmission and plastic-response quantities to supervise intermediate predictions, then examine whether passing those predictions to the field model helps. Their physical interpretation motivates this arrangement; it does not establish its predictive benefit.

The common criterion is the final local accumulated equivalent plastic strain field at unseen operating conditions, with intermediate responses estimated from the permitted inputs. Comparisons under common data and evaluation conditions must establish whether any benefit comes from the proposed supervision or training procedure. More accurate descriptors or a smaller training residual are useful observations, but the engineer still needs the complete map to be accurate.

The main research question therefore concerns how physical auxiliary supervision and staged residual modelling affect that map when FEM data are limited. The field representation and branch alternatives support this investigation by examining how the underlying predictor learns. Chapter 3 turns these remaining uncertainties into research goals; Chapter 4 specifies the models and comparisons needed to answer them.

# 3. Research Goals


The thesis builds on the spatial solder-response prediction demonstrated by Meier et al., reviewed in Section 2.1. Its focus is the contribution of physical auxiliary supervision and staged residual modelling. The present target consists of local accumulated equivalent plastic strain values at 81 prescribed locations; the predecessor used 48 sub-area averages. These different spatial definitions require comparison on a common task before any accuracy advantage can be claimed.

The studied assembly has fixed geometry and a fixed loading protocol, with vibration at 125 Hz. FEM references describe SAC305 solder using an Anand material model. The required output is the accumulated strain field at 0.080 s, for supplied temperature, excitation amplitude and PCB Young's modulus. Intermediate physical responses must be predicted from those inputs when a new condition is queried. The intended engineering use is response comparison; fatigue-life prediction lies outside the required output.

The scope is interpolation at unseen combinations within a FEM-supported domain. 

## 3.1 Main research question and subgoals

Within this scope, the central research question is:

**How do physical auxiliary supervision and staged residual modelling affect the accuracy of local accumulated equivalent plastic strain predictions at unseen operating conditions when FEM training data are limited?**

Four subgoals make this question testable.

1. **Establish the reference prediction.** Determine the accuracy and spatial error patterns of the Plain DeepONet Baseline, trained on the local field alone. This establishes what the available cases support before introducing auxiliary responses or correction. Any claim about an advantage over direct output prediction additionally requires a direct predictor trained and evaluated on the same current task.
2. **Assess the contribution of physical responses.** Determine whether supervising vibration transmission and global plastic response, and passing their predicted descriptors to the field model, improves unseen-condition field accuracy. Controlled comparisons must distinguish auxiliary supervision from changes in information flow and model capacity. Descriptor accuracy helps explain the outcome; the local field remains the criterion.
3. **Assess the predictability of the remaining error.** Determine whether a learned residual correction improves the complete prediction over the initial field predictor. Reconstructing a known residual establishes representational capacity; the required test is whether the correction can be predicted for a withheld condition.
4. **Assess how the training stages interact.** Determine how introducing auxiliary objectives during initial training, and fixing or selectively adjusting the initial predictor during correction training, affect the final field. Comparisons must evaluate the complete predictor and account for fitting variability and computational effort.

Branch alternatives, including spline-based networks, support these goals by probing how the condition–response relationship is learned. Their relevance is judged through the same field-prediction task.

## 3.2 Required outcomes and evaluation criteria

The required technical outcome is a predictor returning 81 nonnegative, dimensionless strain values with their fixed location mapping. Assessment concerns those locations at the stated target time. FEM fields provide the supervised reference; agreement with them establishes surrogate accuracy relative to that numerical model.

The principal accuracy measure is the relative $L_2$ field error for each withheld case: the size of the prediction-error vector divided by the size of the FEM-reference vector. It is evaluated after conversion back to physical strain. Reporting must expose variation across cases and repeated fits, supported by spatial error maps and errors in peak magnitude and location. This connects overall accuracy to the engineer's concern about where deformation concentrates. Exact metric definitions and aggregation rules belong to Section 4.1.7.

Comparisons must use common case membership and declared training budgets, withholding each evaluation case's complete field and auxiliary labels from fitting. Cases repeatedly consulted during development must be distinguished from independent final assessment. The scientific outcome is an evidence-based account of when each proposed change helps, has little effect or worsens prediction. Timing FEM generation, model fitting and complete-field inference on specified hardware must also establish the cost of preparing the surrogate and using it for subsequent queries.

The investigation therefore proceeds from validated FEM cases and a consistent spatial mapping to a common evaluation protocol and reference predictor. Controlled comparisons then address the four subgoals. Additional modulus data enable the broader assessment defined in Section 3.1; until available, stiffness generalisation remains unresolved. The resulting evidence must connect every conclusion to its tested conditions and comparison. Chapter 4 begins by following one query through the prediction process, then specifies the methods and experiments needed to make those comparisons.

# 4. Main Chapter of the Thesis

**To develop:** Introduce the relationship between Methods, Experiments and Results, and Discussion before the first subsection. Organise the work by research question, using tickets as source records for reconstructing decisions and evidence.

## 4.1 Methods

The method can first be understood by following one operating condition from its supplied inputs to the predicted strain map. The worked example below introduces the information flow and the arithmetic before the detailed definitions of data generation, transformations, model variants and evaluation. It describes the current fixed-modulus development configuration; extending the scientific assessment across PCB modulus requires the broader study specified in Chapter 3.

### 4.1.1 A worked example: from an unseen condition to a strain map

Consider the solder region introduced in Chapter 1. Let the query be a temperature and amplitude combination within the FEM-supported development domain, at a PCB modulus of 20 GPa:

$$
\boldsymbol{\mu}_{\star}=(T_{\star},20\,\mathrm{GPa},A_{\star}).
$$

The star marks the condition being predicted; it does not specify a particular measured case. For evaluation, the complete condition and its FEM responses are withheld from fitting. The target is accumulated equivalent plastic strain at 0.080 s at the 81 prescribed spatial locations. The vibration frequency is fixed at 125 Hz. Those locations are selected queries, not a regular Cartesian grid or a volume-integration rule. A contour drawn between them is a visual representation; accuracy between the sampled locations requires separate evidence.

**Start with the information available at prediction time.** The model receives the operating condition and uses the fixed location mapping. It does not receive the query's FEM strain field or any of its FEM response descriptors. The distinction from fitting is explicit:

| Information | Use during fitting | Use at the query condition |
|---|---|---|
| Temperature, PCB modulus and amplitude | Inputs describing each training case | Supplied inputs |
| Fixed spatial coordinates | Identify where each target value belongs | Identify where to return predictions |
| FEM local strain field | Main supervised target | Withheld reference for subsequent evaluation, if available |
| FEM transmission and global plastic-response descriptors | Auxiliary supervised targets | Unavailable to the predictor; estimated by the model |
| Fitted transformations and network parameters | Determined using training membership | Applied without fitting to the query's reference values |

The physical inputs are rescaled before entering the network. Rescaling makes numerical magnitudes suitable for learning while preserving which temperature, modulus and amplitude they represent. The fitted model then evaluates a sequence of functions; it does not repeat the FEM calculation or perform a new training run for this query.

**Predict intermediate responses that may help describe the case.** The first descriptor is the transmission ratio

$$
H_e=\frac{A_e}{A},
$$

where $A_e$ is the edge-response amplitude, obtained from half the range of the exported edge-displacement samples in the final cycle, and $A$ is the imposed source amplitude. This quantity describes the relative amplitude of that observed response. Its value at the query is estimated from the operating inputs.

The global plastic-response descriptor describes the increase in volume-averaged accumulated equivalent plastic strain during the last cycle:

$$
g=\operatorname{EPEQAvg}(0.080\,\mathrm{s})
-\operatorname{EPEQAvg}(0.072\,\mathrm{s}).
$$

The model estimates this descriptor using the operating inputs and its predicted transmission descriptor. The field-prediction branch subsequently receives the operating inputs and both predicted descriptors. Thus the hierarchy adds an information path while retaining the original condition; it does not force the entire field to be determined by two scalars alone. The actual network passes standardised logarithmic representations of the descriptors between these parts. Their physical definitions above explain what the supervised outputs represent.

During training, errors in these descriptor predictions are compared with their FEM labels. This is the meaning of physical auxiliary supervision in the example. It gives selected internal predictions explicit targets but does not enforce the full Anand constitutive equations or prove a physical causal decomposition. The local target is an accumulated field at the final time, whereas $g$ is a volume-averaged increment over one cycle. Averaging the 81 local outputs would neither reproduce its spatial averaging nor its time interval.

The illustrated development variant also passes four learned features from the global-response representation to the field branch and to the later corrector. These form a Latent Augmentation: they have no direct physical target of their own. They may carry information useful for prediction, but their coordinates are not measured damage variables or uniquely identifiable material states.

**Assemble the initial spatial prediction.** For this configuration, the network represents the field in standardised logarithmic coordinates. If $p_j$ is a positive reference strain at location $j$, its transformed value is

$$
z_j=\frac{\ln p_j-m_{\log p}}{s_{\log p}},
$$

where one mean $m_{\log p}$ and one positive population standard deviation $s_{\log p}$ are fitted from the logarithms of all training field values. The same pair is used for every output location and for evaluation cases. Taking the logarithm changes how the numerical representation distinguishes response magnitudes; subtracting the mean and dividing by the scale standardises that representation.

The branch predicts coefficients from the case information, while the trunk evaluates learned spatial functions at the fixed coordinates. Their combination gives

$$
\widehat z_{b,j}=b_0+
\sum_{k=1}^{r} b_k(\boldsymbol{q}_{\star})\,t_k(\boldsymbol{x}_j),
$$

where $\boldsymbol{q}_{\star}$ collects the rescaled operating inputs, predicted descriptor representations and the selected learned features. Both the coefficients and spatial functions are learned. Their roles can be pictured as choosing and combining spatial patterns, but those patterns need not correspond to mechanical modes or have individual physical interpretations.

For a small arithmetic example, show only three locations and two spatial patterns. Let the pattern values be $(1,1,1)$ and $(0,1,2)$, with coefficients 0.4 and 0.2 and zero combination bias. Then

$$
\widehat{\boldsymbol z}_b
=0.4(1,1,1)+0.2(0,1,2)
=(0.4,0.6,0.8).
$$

All numbers in this arithmetic example are invented for explanation, including the later corrections and transformation constants. They are not extracted FEM values, fitted weights or experimental results. The actual development configuration combines 32 spatial functions at all 81 locations; the two-pattern example only makes the summation visible.

**Predict a correction using information already available.** In the fixed-modulus development configuration, the corrector receives scaled temperature and amplitude together with the four learned features, standardised using their training-case values. It predicts 81 coefficients $\boldsymbol c$. A fixed, invertible 81-by-81 Full-Node Output-Coordinate Map $B$ converts those coefficients into corrections at the output locations:

$$
\widehat{\boldsymbol z}_{\mathrm{final}}
=\widehat{\boldsymbol z}_b+B\boldsymbol c.
$$

This map changes the coordinates in which the correction is learned without reducing the number of representable field dimensions. It does not establish that a particular residual can be predicted accurately at an unseen condition. Suppose the resulting correction $B\boldsymbol c$ at the three displayed locations is $(0.1,-0.05,0.2)$. The corrected transformed values are then $(0.5,0.55,1.0)$.

**Convert the result back to the requested strain values.** The inverse transformation is

$$
\widehat p_j
=\exp\!\left(m_{\log p}
+s_{\log p}\widehat z_{\mathrm{final},j}\right).
$$

For the illustrative constants $m_{\log p}=\ln(0.01)$ and $s_{\log p}=1$, the complete calculation at the displayed positions is:

| Displayed location | Initial $\widehat z_b$ | Predicted correction | Final $\widehat z$ | Final strain $0.01\exp(\widehat z)$, rounded |
|---|---:|---:|---:|---:|
| 1 | 0.40 | +0.10 | 0.50 | 0.016487 |
| 2 | 0.60 | $-0.05$ | 0.55 | 0.017333 |
| 3 | 0.80 | +0.20 | 1.00 | 0.027183 |

The correction is additive in the transformed field, not in physical strain. Equivalently, it multiplies the initial physical prediction at location $j$ by $\exp(s_{\log p}[B\boldsymbol c]_j)$. This explains how a negative correction reduces the middle value while positive corrections increase the others. No FEM reference has been supplied for this toy calculation, so it demonstrates how a prediction is formed, not whether the correction improves it.

**Relate the prediction to the training stages.** The current development configuration first trains the hierarchical predictor, beginning with a field-only warm-up and progressively introducing descriptor supervision. A later stage introduces the corrector with zero initial output, so its initial contribution to the field is zero. In that stage, the descriptor networks and learned-feature projection remain fixed, while the branch, trunk and combination bias continue to adjust together with the corrector. The complete initial predictor is therefore not a Frozen Base in this configuration. Its changing field and the changing correction jointly determine the final prediction. The detailed training protocol must specify these choices and the objective used in each stage; their benefit is an empirical question.

**Assess the completed prediction.** After the prediction is formed, its 81 physical strain values can be compared with the withheld FEM field. The comparison asks both how large the complete-field error is and where the error occurs. To evaluate the contribution of auxiliary supervision, the relevant comparator is the Plain DeepONet Baseline under a declared common protocol, supplemented by controls that separate changes in information and capacity. To evaluate a consistency constraint, a Matched Multitask Control retains the same auxiliary supervision and prediction structure while omitting that constraint. To evaluate the correction and training stages, controlled alternatives must be compared as complete predictors.

The running case makes these comparisons understandable, but the evidence must include all declared evaluation cases, repeated fits and difficult cases as well. Cases repeatedly consulted during model development cannot serve as untouched final confirmation. Agreement on the current fixed-modulus data also leaves stiffness generalisation unresolved. These requirements connect the prediction procedure back to the thesis question: which additions improve the field that the engineer actually needs, under which tested conditions?

### 4.1.2 Physical problem and FEM reference generation

**To develop:** Document the assembly geometry, mesh, constraints, excitation, temperature treatment, PCB material properties, SAC305 Anand formulation and actual parameter set, solver and time integration. Add convergence and available experimental validation. Define local accumulated strain and any cycle-increment or volume-averaged quantities separately.

### 4.1.3 Dataset construction and parameter coverage

**To develop:** Explain extraction, location mapping, units and quality checks. Record each dataset and its parameter coverage, including newly added modulus values when available. Define one complete FEM case as the unit of splitting and evaluation.

### 4.1.4 Preprocessing and field representation

For strictly positive strain targets, predicting a logarithmic quantity and recovering the physical value through an exponential produces positive outputs. This addresses the sign issue identified in the preceding study. Exact zeros require separate treatment. The construction imposes positivity; spatial accuracy and mechanical consistency still require their own assessment. Section 4.1.1 illustrates the logarithmic transformation and its inverse for the current development configuration.

**To develop:** Complete the specification of input scaling, target transformations and inverse transformations. Fit data-dependent preprocessing on training cases only. Explain spatial query weighting and the physical meaning of each reported metric.

### 4.1.5 Baseline and physically supervised models

**To develop:** Present the baseline and final investigated architecture with diagrams and tensor definitions. Describe the predicted transmission and global plastic-response quantities, their supervision, and the information available during inference. Separate physically labelled descriptors from unsupervised latent features. For the branch comparisons introduced in Section 2.2.5, specify the MLP and KAN configurations, spline degree and grids, any coarse-to-fine transfer, and the components trained in each phase.

### 4.1.6 Residual correction and staged training

**To develop:** Define the correction target, its input information and spatial representation. Specify which components are fixed or trainable in each stage, the losses, schedules and optimisation settings. Distinguish reduced spatial representations from a full-rank change of output coordinates.

### 4.1.7 Evaluation protocol and reproducibility

Evaluation asks whether a modelling change improves the complete strain field at a new operating condition. The protocol must therefore preserve the information available at prediction time, examine the physical output and make the proposed change distinguishable from other differences between models.

**Withhold complete operating conditions.** Return to the engineer requesting a plastic strain map at a new temperature–amplitude–modulus combination. Evaluation should reproduce that situation: the complete condition and its response field must be withheld from fitting. Many spatial values from one simulation describe one field; they are not independent samples of the operating domain.

The same boundary applies to auxiliary labels. A withheld condition's intermediate FEM responses cannot be supplied to the predictor when the intended use requires estimating them. Preprocessing transformations and any data-derived spatial representation must also be fitted using only the permitted training data.

Model selection introduces another distinction. Validation data guide choices such as network size, loss weights and training duration. A final assessment evaluates the selected procedure. Cawley and Talbot show how repeated selection against a finite evaluation criterion can itself produce optimistic estimates. Consequently, cases repeatedly used to choose a model belong to development evaluation, even when they were withheld from a particular fit. [[19]](<https://www.jmlr.org/papers/v11/cawley10a.html>)

**Inspect the field beyond the fitting objective.** An average error compresses many discrepancies into one number. For example, the mean squared error in physical strain coordinates is

$$
\operatorname{MSE}_{p}
=\frac{1}{NM}\sum_{i=1}^{N}\sum_{j=1}^{M}
\left[
\widehat{p}(\boldsymbol{\mu}_i,\boldsymbol{x}_j)
-p(\boldsymbol{\mu}_i,\boldsymbol{x}_j)
\right]^2.
$$

Here, $N$ is the number of evaluated cases and $M$ is the number of output locations. The indices $i$ and $j$ identify a case and a location, respectively; $\boldsymbol{\mu}_i$ is its operating condition and $\boldsymbol{x}_j$ is the queried position. The values $p$ and $\widehat p$ are the FEM reference strain and the predicted strain after any inverse transformation. Large absolute discrepancies have a strong influence on this quantity. Fitting a logarithmic target changes that emphasis because differences of logarithms describe ratios between positive values.

The fitting objective therefore does not replace inspection of the intended physical output. A low average error can coexist with a misplaced maximum or a poor prediction in one operating region. Evaluation should expose how errors vary over cases and locations, including difficult cases, while reporting the field in physical strain units after any inverse transformation.

**Isolate the contribution of each change.** Adding physical supervision changes the available labels and may change model capacity. Adding a corrector changes the number of parameters and the training procedure. Repeated fits and appropriately matched comparisons help determine whether an observed difference is associated with the intended change or with fitting variability. Section 4.2.1 specifies how these principles apply to the individual experiments.

**Measure computational costs.** Computational benefit requires measurements of data-generation, training and prediction costs. These timings must identify the hardware and workload so that preparation costs and the cost of each subsequent prediction can be interpreted together.

**To develop:** Specify fitting, model-selection and final-evaluation case membership within the scope defined in Section 3.1; seeds; comparison budgets; the exact reported metrics, including spatial and peak-related measures; and timing procedures. Record enough configuration to interpret each completed experiment. State how final confirmation will be separated from iterative development. The principles and example error measure above do not yet constitute the complete experimental protocol.

## 4.2 Experiments and Results

### 4.2.1 Experimental design and common comparisons

**To develop:** Map each experiment to a Chapter 3 subgoal. State its hypothesis, common data and settings, deliberate intervention and measured outcomes. Specify repeated fits and matched controls under the protocol in Section 4.1.7, identifying differences in supervision, model capacity and training procedure. Keep comparisons across changing datasets separate.

### 4.2.2 Baseline performance and spatial error patterns

**To develop:** Report errors for each operating condition, together with representative reference, prediction and error contours. Include peak-related measures where relevant and variation across repeated fits. Establish the baseline before discussing modifications.

### 4.2.3 Contribution of physical auxiliary supervision

**To develop:** Present matched comparisons for descriptors, hierarchical information paths and any consistency constraints. Report main-field performance alongside auxiliary accuracy; include neutral and negative outcomes needed to answer the research question.

### 4.2.4 Contribution of residual correction and training stages

**To develop:** Compare the initial and complete predictors, relevant correction representations, and the principal stagewise-training choices. Distinguish ideal reconstruction diagnostics from predictive corrector results.

### 4.2.5 Generalisation across temperature, amplitude and PCB modulus

**To develop:** Evaluate withheld parameter combinations using the expanded data when ready. Report parameter coverage and where evidence remains limited. Keep fixed-modulus findings identifiable as such.

### 4.2.6 Computational performance and supporting studies

**To develop:** Report FEM-reference generation, fitting and inference costs on specified hardware. Include activation, feature or uncertainty studies only when they clarify a main research question; otherwise summarise them briefly or place details in an appendix if the thesis format allows.

## 4.3 Discussion

### 4.3.1 Answers to the research questions

**To develop:** Interpret the evidence against each stated goal. Distinguish measured findings from mechanistic explanations and hypotheses.

### 4.3.2 Relationship to existing research

**To develop:** Revisit the closest studies from Chapter 2. Explain agreements and differences in terms of inputs, outputs, physical information, datasets and evaluation conditions.

### 4.3.3 Limitations and engineering implications

Agreement with held-out FEM references assesses fidelity to the selected numerical model. Physical validity requires separate evidence about that FEM model and the assembly it represents.

**To develop:** Interpret the completed results within the parameter coverage defined in Section 3.1. Discuss FEM-reference validity, finite sampling, repeated model selection, spatial resolution and the limits of physical interpretation. Keep fixed-modulus findings distinct from any evidence obtained across PCB modulus. Explain what response-analysis use is supported by the final evidence, including the measured computational costs reported in Section 4.2.6.

# 5. Summary and Outlook

**To develop:** Give a concise account of the problem, approach and supported findings after the study is complete. Introduce no new experimental claims.

## 5.1 Summary and main conclusions

**To develop:** Answer the research questions directly, including where an investigated method did not improve the desired outcome. Relate the findings to the intended engineering use.

## 5.2 Outlook

**To develop:** Identify future work justified by the findings, such as richer sampling, additional material or loading conditions, independent validation or subsequent reliability applications. Distinguish work required for the present thesis from longer-term extensions.

# 6. References

[1] Meier, Karsten; Qi, Yichen; Albrecht, Oliver; Garzón, Cristian A.; Bock, Karlheinz (2025). [Advancing Electronic Package Reliability Analysis by Predicting Solder Joint Strain Patterns Using Neural Networks](<https://doi.org/10.1109/EPTC67330.2025.11392225>). 2025 IEEE 27th Electronics Packaging Technology Conference (EPTC), pp. 1–8. Added to IEEE Xplore 2026-02-24.

[2] Anand, L. (1985). [Constitutive equations for hot-working of metals](<https://doi.org/10.1016/0749-6419(85)90004-X>). International Journal of Plasticity, 1(3), 213–231.

[3] Goodfellow, Ian; Bengio, Yoshua; Courville, Aaron (2016). [Deep Learning](<https://www.deeplearningbook.org/>). MIT Press.

[4] Lu, L.; Jin, P.; Pang, G.; Zhang, Z.; Karniadakis, G. E. (2021). [Learning nonlinear operators via DeepONet based on the universal approximation theorem of operators](<https://doi.org/10.1038/s42256-021-00302-5>). Nature Machine Intelligence, 3, 218–229.

[5] Koric, Seid; Viswantah, Asha; Abueidda, Diab W.; Sobh, Nahil A.; Khan, Kamran (2024). [Deep learning operator network for plastic deformation with variable loads and material properties](<https://doi.org/10.1007/s00366-023-01822-x>). Engineering with Computers, 40(2), 917–929.

[6] Liu, Ziming; Wang, Yixuan; Vaidya, Sachin; Ruehle, Fabian; Halverson, James; Soljačić, Marin; Hou, Thomas Y.; Tegmark, Max (2025). [KAN: Kolmogorov–Arnold Networks](<https://proceedings.iclr.cc/paper_files/paper/2025/hash/afaed89642ea100935e39d39a4da602c-Abstract-Conference.html>). International Conference on Learning Representations (ICLR). First circulated as arXiv:2404.19756 in 2024.

[7] Abueidda, Diab W.; Pantidis, Panos; Mobasher, Mostafa E. (2025). [DeepOKAN: Deep operator network based on Kolmogorov Arnold networks for mechanics problems](<https://doi.org/10.1016/j.cma.2024.117699>). Computer Methods in Applied Mechanics and Engineering, 436, 117699.

[8] Raissi, M.; Perdikaris, P.; Karniadakis, G. E. (2019). [Physics-informed neural networks: A deep learning framework for solving forward and inverse problems involving nonlinear partial differential equations](<https://doi.org/10.1016/j.jcp.2018.10.045>). Journal of Computational Physics, 378, 686–707.

[9] Wang, S.; Wang, H.; Perdikaris, P. (2021). [Learning the solution operator of parametric partial differential equations with physics-informed DeepONets](<https://doi.org/10.1126/sciadv.abi8605>). Science Advances, 7(40), eabi8605.

[10] He, J.; Pal, D.; Najafi, A.; Abueidda, D.; Koric, S.; Jasiuk, I. (2024). [Material-Response-Informed DeepONet and Its Application to Polycrystal Stress–Strain Prediction in Crystal Plasticity](<https://doi.org/10.1007/s11837-024-06681-5>). JOM, 76, 5744–5754.

[11] Chen, Z.; Badrinarayanan, V.; Lee, C.-Y.; Rabinovich, A. (2018). [GradNorm: Gradient Normalization for Adaptive Loss Balancing in Deep Multitask Networks](<https://proceedings.mlr.press/v80/chen18a.html>). Proceedings of the 35th International Conference on Machine Learning, 80, 794–803.

[12] Yu, T.; Kumar, S.; Gupta, A.; Levine, S.; Hausman, K.; Finn, C. (2020). [Gradient Surgery for Multi-Task Learning](<https://proceedings.neurips.cc/paper/2020/hash/3fe78a8acf5fda99de95303940a2420c-Abstract.html>). Advances in Neural Information Processing Systems, 33.

[13] Friedman, Jerome H. (2001). [Greedy function approximation: A gradient boosting machine](<https://doi.org/10.1214/aos/1013203451>). The Annals of Statistics, 29(5), 1189–1232.

[14] Schaback, Robert; Wendland, Holger (2006). [Kernel techniques: From machine learning to meshless methods](<https://doi.org/10.1017/S0962492906270016>). Acta Numerica, 15, 543–639.

[15] Reverberi, Antoine; Procario, Jennifer (2024). [Explaining SimAI: How AI is Applied to Numerical Simulation In Practice](<https://ansys.synopsys.com/blog/explaining-simai>). Ansys Blog. Published 2024-05-13; accessed 2026-09-24.

[16] Ansys, Inc. (n.d.). [Ansys SimAI Overview](<https://ansyshelp.ansys.com/public/Views/Secured/SimAI/v000/en/SimAI_ug/SimAI_ug/C_UG_SAI_overview.html>). SimAI User's Guide, SimAI Premium. Accessed 2026-09-24; publication date not stated.

[17] Wang, S.; Teng, Y.; Perdikaris, P. (2021). [Understanding and Mitigating Gradient Flow Pathologies in Physics-Informed Neural Networks](<https://doi.org/10.1137/20M1318043>). SIAM Journal on Scientific Computing, 43(5), A3055–A3081.

[18] Ansys, Inc. (n.d.). [Training Data Selection Best Practices](<https://ansyshelp.ansys.com/public/Views/Secured/SimAI/v000/en/SimAI_ug/SimAI_ug/C_UG_SAI_training_data_selection_best_practices.html>). SimAI User's Guide. Accessed 2026-09-24; publication date not stated.

[19] Cawley, G. C.; Talbot, N. L. C. (2010). [On Over-fitting in Model Selection and Subsequent Selection Bias in Performance Evaluation](<https://www.jmlr.org/papers/v11/cawley10a.html>). Journal of Machine Learning Research, 11(70), 2079–2107.

[^1]: 
