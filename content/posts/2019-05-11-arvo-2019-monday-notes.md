---
layout: post
title: "ARVO 2019 Monday Notes"
author: Peter Boone
tags: ["choroideremia", "ARVO"]
date: 2019-05-11
---

![east vancouver convention center](/imgs/arvo-2019/2019-04-28-east-van-convention-center.jpg)

_The Vancouver Convention Center's East Building_

These are my notes from the second day at ARVO 2019. Check out [Sunday's notes](https://boonepeter.github.io/2019/05/11/arvo-2019-sunday-notes.html) if you missed them. 

## Changing What it Means to be Blind: We're All in This Together
This session took place late in the evening and was put together by [Kristin Smedley](http://crb1.org/) from the Curing Retinal Blindness Foundation. Smedley began the session by playing a TEDx talk she gave in 2017 at Lincoln Square. You can find her talk on the [TEDxLincolnSquare website](https://www.tedxlincolnsquare.com/).

Second in the lineup was [Gina Harper](https://www.davisenterprise.com/sports/for-gena-harper-its-more-than-meets-the-eye/). Gina is a financial consultant at Morgan Stanley and was born with congenital glaucoma that has rendered her almost completely blind. Her talk was extremely frank and focused on what it takes to live and thrive with blindness. She emphasized that you cannot throw a pity party for yourself for very long, but that you need to "buck up" and work through the hard times. 

[Avril Daly](http://www.retina-international.org/our-story/meet-our-ceo/), the CEO of Retina International, spoke about the importance of coordinating with other rare disease organizations. Rare diseases that are not related to the eye can still be valuable allies since they are working towards very similar goals. Banding together to have a larger influence on public policy and research funding is a wise strategy that we should follow. 

Next [Chris Moen](https://www.curechm.org/about-the-foundation/chris-moen-md-president) from the Choroideremia Research Foundation spoke about shifting the way researchers and ophthalmologists view blind patients. He made a great point in showing what disease looks like to the different people who work on it. To a researcher it looks like a mouse model, some iPSCs, or a genetic test. To an ophthalmologist it looks like an OCT of FAF scan. To an optometrist it looks like a visual acuity test. But in reality, the disease is a person, he is a father, a doctor, a husband, and a friend. The human aspect of the doctor-patient interaction needs to be emphasized, and Chris encouraged clinicians to ask how the patient really is doing and refer them on to low vision specialists or counselors to help with problems beyond their scope.

## Paper Sessions
#### The combination of biologic-loaded liposomes and retinal ganglion cell transplant rescues retinal function in an NMDA toxicity model - *Anne Zebitz Zebitz Eriksen*
This group used a combination treatment - delivering both cell transplantation and a neuroprotective growth factor. IGF-1 and CNTF were the growth factors delivered. They found a 1.5 fold increase in surviving host RGCs with both treatments compared to cell transplantation only. The growth factors did not affect the survival of the donor RGCs. In their NMDA model 75% of RGCs die.

#### In Vivo delivery of CRISPR-Cas9 complex to the retina via a cell penetrating peptide - *Vanessa Yanez*
They injected a cell penetrating peptide nicknamed "POD" along with the CRISPR-Cas9 complex into the vitreous. They found Cas9-GFP in the ONL, INL, and GCL of the retina of mice. They observed more efficient uptake with subretinal injections particularly in the RPE. 

#### Machine learning based end-to-end pipeline for OCT angiography of diabetic retinopathy - *Morgan Heisler*
This study used cascaded deep neural networks for the segmentation of 5 retinal layers, intraretinal fluid, microvasculature, the foveal avascular zone, and diabetic retinopathy severity grading. They used UNET architecture. The classification of negative to severe diabetic retinopathy was performed separately. They found the following Dice indexes:
|layer|Dice index|
|---|---|
|RNFL|0.942|
|GCL|0.957|
|INL|0.968|
|OPL|0.959|
|fluid|0.786|
|microvasculature|0.836|
|FAZ|0.957|



## Posters
#### Automated cone identification in adaptive optics retinal images of CHM using a convolutional neural network - *Jessica Morgan*
An open sourced network [Cunefare *Scientific Reports* 2017](https://www.ncbi.nlm.nih.gov/pubmed/28747737) that was trained on normal AO images was used and retrained. The purpose of this method is to speed up the segmentation of AO images. The CHM-trained network had a higher true positive rate than the original, but also had a higher mean false discovery rate. Morgan said the network only takes in split detection images. It would be great to use coregistered split detection and confocal images to help with detection. Another poster presented work on this, which I will link to it [here] when I find it in my notes.
#### A deep convolutional neural network model for automatic measurement of ellipsoid zone width in RP patients - *Yi-Zhong Wang*
A lot of automated ellipsoid zone identification software doesn't work well in diseased retinas. 45 epochs for training. Accuracy was 97%. Correlation between EZ width measured by model and by manual graders was 0.96. A mean difference of 0.20 mm between the two was found. 

#### Deep learning algorithm for diagnosis of Alzheimer's disease using multimodal retinal imaging - *Clayton Ellis Wisely*
Their algorithm demonstrated AD diagnosis with close to 80% accuracy

#### Clinical validation of a machine learned algorithm for detection of diabetic retinopathy and diabetic macular edema in fundus images - *Kira Whitehouse*
Detected referable DR with a sensitivity of 90.4% and specificity of 97.5%, classified 1 of 130 as less than moderate which graders classified as proliferative DR, classified 19 of 325 as no DME which graders classified as vision threatening.

#### Understanding the relationship between ellipsoid zone inner border variation and its effect on visual acuity in diabetic retinopathy patients - *Jae Kim*
This work examines the variability in the ellipsoid zone, how "wavy" it is. The algorithm to find the variability predicted a worsening visual acuity based on OCT images in 85.4% of one of the groups. This could be used as an additional tool to determine visual prognosis in a clinical setting. 

[Tuesday's Notes](https://boonepeter.github.io/2019/05/12/arvo-2019-tuesday-notes.html)




