# B2CPolicies

## Read User by Username/EMail

http://graph.windows.net/TenantId/users?api-version=1.6&$filter=signInNames/any(x:x/value eq 'UserId')

## Login through username

## Add this technical profile to TrustFrameworkExtensions.xml to write the user to Azure AD

<TechnicalProfile Id="AAD-UserWriteUsingLogonName">
    <Metadata>
        <Item Key="Operation">Write</Item>
        <Item Key="RaiseErrorIfClaimsPrincipalAlreadyExists">true</Item>
    </Metadata>
    <InputClaims>
        <InputClaim ClaimTypeReferenceId="signInName" PartnerClaimType="signInNames.userName" Required="true" />
    </InputClaims>
    <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="signInName" PartnerClaimType="signInNames.userName" />
        <PersistedClaim ClaimTypeReferenceId="email" PartnerClaimType="strongAuthenticationEmailAddress" />
        <PersistedClaim ClaimTypeReferenceId="newPassword" PartnerClaimType="password" />
        <PersistedClaim ClaimTypeReferenceId="displayName" DefaultValue="SomeDefaultDisplayNameValue" />
    </PersistedClaims>
    <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="objectId" />
        <OutputClaim ClaimTypeReferenceId="newUser" PartnerClaimType="newClaimsPrincipalCreated" />
        <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
        <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    </OutputClaims>
    <IncludeTechnicalProfile ReferenceId="AAD-Common" />
    <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
</TechnicalProfile>

## Add this technical profile to TrustFrameworkExtensions.xml that is called by the user journey. The critical change here is the <Item Key="LocalAccountType">Username</Item> and <Item Key="LocalAccountProfile">true</Item>

<TechnicalProfile Id="LocalAccountSignUpWithLogonName">
    <DisplayName>User ID signup</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    <Metadata>
        <Item Key="IpAddressClaimReferenceId">IpAddress</Item>
        <Item Key="ContentDefinitionReferenceId">api.localaccountsignup</Item>
        <Item Key="LocalAccountType">Username</Item>
        <Item Key="LocalAccountProfile">true</Item>
        <Item Key="language.button_continue">Create</Item>
    </Metadata>
    <CryptographicKeys>
        <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
    </CryptographicKeys>
    <InputClaims>
        <InputClaim ClaimTypeReferenceId="signInName" />
    </InputClaims>
    <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="objectId" Required="true" />
        <OutputClaim ClaimTypeReferenceId="signInName" Required="true" />
        <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
        <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
        <OutputClaim ClaimTypeReferenceId="email" Required="true" />
        <OutputClaim ClaimTypeReferenceId="executed-SelfAsserted-Input" DefaultValue="true" />
        <OutputClaim ClaimTypeReferenceId="newUser" />
        <OutputClaim ClaimTypeReferenceId="authenticationSource" />
        <OutputClaim ClaimTypeReferenceId="userPrincipalName" />
    </OutputClaims>
    <ValidationTechnicalProfiles>
        <ValidationTechnicalProfile ReferenceId="AAD-UserWriteUsingLogonName" />
    </ValidationTechnicalProfiles>
    <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
</TechnicalProfile>


## Sigin Technical Profile for Reference

<TechnicalProfile Id="SelfAsserted-LocalAccountSignin-Username">
    <DisplayName>Local Account Signin</DisplayName>
    <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
    <Metadata>
        <Item Key="SignUpTarget">SignUpWithLogonUsernameExchange</Item>
        <Item Key="setting.operatingMode">Username</Item>
        <Item Key="ContentDefinitionReferenceId">api.selfasserted</Item>
    </Metadata>
    <IncludeInSso>false</IncludeInSso>
    <InputClaims>
        <InputClaim ClaimTypeReferenceId="signInName" />
    </InputClaims>
    <OutputClaims>
        <OutputClaim ClaimTypeReferenceId="signInName" Required="true" />
        <OutputClaim ClaimTypeReferenceId="password" Required="true" />
        <OutputClaim ClaimTypeReferenceId="objectId" />
        <OutputClaim ClaimTypeReferenceId="authenticationSource" />
    </OutputClaims>
    <ValidationTechnicalProfiles>
        <ValidationTechnicalProfile ReferenceId="login-NonInteractive" />
    </ValidationTechnicalProfiles>
    <UseTechnicalProfileForSessionManagement ReferenceId="SM-AAD" />
</TechnicalProfile>


## Add this user journey to TrustFrameworkExtensions.xml

<UserJourney Id="SignUpOrSignIn">
    <OrchestrationSteps>
        <OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
            <ClaimsProviderSelections>
                <ClaimsProviderSelection ValidationClaimsExchangeId="LocalAccountSigninUsernameExchange" />
            </ClaimsProviderSelections>
            <ClaimsExchanges>
                <ClaimsExchange Id="LocalAccountSigninUsernameExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Username" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="2" Type="ClaimsExchange">
            <Preconditions>
                <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
                    <Value>objectId</Value>
                    <Action>SkipThisOrchestrationStep</Action>
                </Precondition>
            </Preconditions>
            <ClaimsExchanges>
                <ClaimsExchange Id="SignUpWithLogonUsernameExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonName" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <!-- This step reads any user attributes that we may not have received when in the token. -->
        <OrchestrationStep Order="3" Type="ClaimsExchange">
            <ClaimsExchanges>
                <ClaimsExchange Id="AADUserReadWithObjectId" TechnicalProfileReferenceId="AAD-UserReadUsingObjectId" />
            </ClaimsExchanges>
        </OrchestrationStep>
        <OrchestrationStep Order="4" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
    </OrchestrationSteps>
    <ClientDefinition ReferenceId="DefaultWeb" />
</UserJourney>


thanks to Omer for his help!


        
        
        
        
