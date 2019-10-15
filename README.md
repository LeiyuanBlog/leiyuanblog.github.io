if (0>=stock
		&&"1".equals(rsrvTxt9)
		&&null!=price1&&!price1.equals("")
		&&null!=price2&&!price2.equals("")
		&&null!=promoDateTo&&!promoDateTo.equals("")
		&&null!=promoDateFrom&&!promoDateFrom.equals("")
		&&((Long.valueOf(promoDateTo)-Long.valueOf(promoDateFrom))>=7776000000L)) {
	return "缺货-长期促销"; 
} else if (0>=stock
		&&"1".equals(rsrvTxt9)
		&&null!=price1&&!price1.equals("")
		&&null!=price2&&!price2.equals("")
		&&null!=promoDateTo&&!promoDateTo.equals("")
		&&null!=promoDateFrom&&!promoDateFrom.equals("")
		&&((Long.valueOf(promoDateTo)-Long.valueOf(promoDateFrom))<7776000000L)) {
	return "缺货-降价"; 
} else if (0>=stock
		&&"1".equals(rsrvTxt9)
		&&null!=promoDescription&&!promoDescription.equals("")
		&&(-1!=promoDescription.indexOf("多")||-1!=promoDescription.indexOf("满"))) {
	return "缺货-多买多省"; 
} else if ("1".equals(rsrvTxt9)
		&&null!=price1&&!price1.equals("")
		&&null!=price2&&!price2.equals("")
		&&null!=promoDateTo&&!promoDateTo.equals("")
		&&null!=promoDateFrom&&!promoDateFrom.equals("")
		&&((Long.valueOf(promoDateTo)-Long.valueOf(promoDateFrom))>=7776000000L)) {
	return "长期促销"; 
} else if ("1".equals(rsrvTxt9)
		&&null!=price1&&!price1.equals("")
		&&null!=price2&&!price2.equals("")
		&&null!=promoDateTo&&!promoDateTo.equals("")
		&&null!=promoDateFrom&&!promoDateFrom.equals("")
		&&((Long.valueOf(promoDateTo)-Long.valueOf(promoDateFrom))<7776000000L)) {
	return "降价"; 
}  else if ("1".equals(rsrvTxt9)
		&&null!=promoDescription&&!promoDescription.equals("")
		&&(-1!=promoDescription.indexOf("多")||-1!=promoDescription.indexOf("满"))) {
	return "多买多省"; 
} else if (0>=stock) {
	return "缺货-普通"; 
} else {
	return "普通"; 
}
