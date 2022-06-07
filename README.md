# Unipro tAccession ID to GeneSymbol in R

아래 코드는 R에서 실행하는 것을 전제로 합니다.  
두가지 패키지의 사전 설치가 필요합니다.  
- Tidyverse
- httr

Input data의 컬럼 "Accession" 이 있어야 하며, 해당 컬럼에 Uniprot Accession ID가 적혀 있어야 합니다. 
ex) E9Q447, Q8C8R3-4 등, "-"로 구분된 Isoform label이 있어도 무관합니다  

사전에 준비된 데이터 프레임의 이름은  
**Uniprot2Gene** 입니다. 컬럼 이름 Accession 에 Uniprot Accession이 포함되어 있습니다.

## 코드 시작

```R
library("tidyverse")
library("httr")

head(Uniprot2Gene)

entry <- str_split(Uniprot2Gene$Accession,"-") %>% as.data.frame
Uniprot2Gene$Entry <- t(entry[1,])[,1] %>% unname
```
코드 설명:  Accession 컬럼에 있는 값을 가져와서 - 로 나눠 Isoform 구분자를 제거한 뒤 Accession ID만 남깁니다.


```R
results <- POST(url = "https://www.uniprot.org/uploadlists/",
                body = list(from = 'ID',
                            to = 'GENENAME',
                            format = 'tab',
                            query = paste(Uniprot2Gene$Entry, collapse = ' ')))

MappingTable <- content(results, type = 'text/tab-separated-values', 
                           col_names = TRUE, 
                           col_types = NULL, 
                           encoding = "UTF-8")
colnames(MappingTable) <- c("Entry","GeneSymbol")
```
코드 설명: Uniprot 사이트를 통해 Uniprot2Gene 데이터프레임의 Entry 컬럼에 들어있는 uniprot accession No. 를 검색 & GeneName을 tab으로 나눠 받아옵니다
그 후 데이터를 MappingTable 변수에 받아온 다음에, join을 위해 컬럼 이름을 Entry와 GeneSymbol로 변경해 받아옵니다.


```R
length(unique(MappingTable$Entry))
duplicate <- MappingTable[which(duplicated(MappingTable$Entry)),]$GeneSymbol
UniqueMappingTable <- MappingTable %>% filter(! GeneSymbol %in% duplicate)
nrow(UniqueMappingTable)
Uniprot2Gene.out <- left_join(Uniprot2Gene,UniqueMappingTable, by="Entry")

head(Uniprot2Gene.out,2)
```
코드 설명: MappingTable은 한개의 UniprotAccession이 여러개의 GeneName을 가지고 있는 경우가 있을 수 있어, 이를 제거해 주기 위해
MappingTable에 AccessionID에서 GeneSymbol이 매칭된 고유 개수를 확인하여 준 뒤  
중복되는 entry에 해당되는 컬럼을 duplicate 변수에 받아옵니다.

duplicate에 해당하는 GeneSymbol을 제거하여 UniqueMappingTable 변수에 데이터를 받아오게 되면
중복이 제거되어 1개의 Accession ID가 1개의 GeneName을 가지게 된 데이터프레임이 남게 됩니다.

UniqueMappingTable 의 길이와 제일 처음 확인한 MappingTable의 Entry개수가 동일하다면, 다음 단계 진행에 문제가 없습니다.

마지막으로 left_join 을 활용하여 Uniprot2Gene 테이블과 UniqueMappingTable 테이블을 "Entry" 를 키값으로 하여 하나로 합쳐 줍니다. 

끗!






