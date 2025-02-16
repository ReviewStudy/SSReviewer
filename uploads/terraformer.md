
https://github.com/GoogleCloudPlatform/terraformer

기존의 생성되어있는 인프라를 테라폼 소스로 가져오는 오픈소스 툴이다!

```
terraformer import
```

와 같은 명령어를 통해서 현재의 인프라를 가져올 수 있다

```
terraformer import aws --resources=* --path-pattern="{output}/" --connect=true --regions=ap-northeast-2 --profile ""
```

이런식으로 실행을 하면 모든 리소스의 파일들을 테라폼으로 가져올 수 있다


### 어떻게 작동을 하는가?
- Provider의 refresh method를 이용해서 모든 데이터를 가져온다
- 이 데이터를 go struct의 구조체로 변경한다
- HCL 파일과 json파일들을 만든다 
- tfstate 파일을 만든다

### Terraform Refresh?
- terraform state 파일을 Terraform이 관리하는 인프라의 현재 상태와 동기화한다
- 필요한 이유로는 테라폼 코드를 이용하여 blueprint를 만들어놨는데 수동으로 누가 수정할 수 가 있다
- Terraform Refresh는 이러한 직접 수정 사항을 반영하기 위해서 terraform state 파일을 고쳐서 격차를 해소를 하기 위함이다

> 테라폼 리프레시의 주요 목적은 구성 파일에 정의된 대로 리소스의 실제 상태와 원하는 상태 간의 드리프트를 감지

This won't modify your real remote objects, but it will modify the [Terraform state](https://developer.hashicorp.com/terraform/language/state).

즉 인프라는 가만히 냅두고 State file만 변경을 하는것이기 때문의 현재 나의 인프라에 있는 모든 리소스들을 드리프트로 감지를 하고 이를 state file로 가져온다는것이다!
이를 이용하여 Terraformer는 테라폼 파일들을 생성하게 된다

물론 실제로 테라폼에서 적용을 할때는 terraform plan, terraform apply 명령어를 수행할때 자동으로 수행하기 때문에 필요는 없다

```
func Import(provider terraformutils.ProviderGenerator, options ImportOptions, args []string) error {

	// provider 객체와 options을 초기화하고, providerWrapper를 생성
	providerWrapper, options, err := initOptionsAndWrapper(provider, options, args)
	if err != nil {
		return err
	}
	defer providerWrapper.Kill() // 함수 종료 시 providerWrapper를 정리

	// provider를 기반으로 ProviderMapping 객체 생성
	providerMapping := terraformutils.NewProvidersMapping(provider)

	// provider에서 모든 서비스의 리소스를 초기화
	err = initAllServicesResources(providerMapping, options, args, providerWrapper)
	if err != nil {
		return err
	}

	// provider에서 현재 리소스 상태를 Refresh
	err = terraformutils.RefreshResourcesByProvider(providerMapping, providerWrapper)
	if err != nil {
		return err
	}

	// Terraform 상태 파일을 변환하여 providerMapping에 반영
	providerMapping.ConvertTFStates(providerWrapper)

	// 리소스별로 추가 데이터를 포함하여 구조체를 정리
	providerMapping.CleanupProviders()

	// import plan을 기반으로 리소스 가져오기 실행
	err = importFromPlan(providerMapping, options, args)

	return err // 최종적으로 발생한 에러 반환
}

```


```
sudo terraformer import aws --resources="ecs" --path-pattern="{output}/" --regions=ap-northeast-2 --profile ""
```

이렇게 리소스를 가져오면 

![[Pasted image 20250217001044.png]]

이렇게 나의 현재 리소스들을 가져온다
