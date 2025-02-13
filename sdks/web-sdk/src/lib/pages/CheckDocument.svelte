<script lang="ts">
  import { T } from '../contexts/translation';
  import { Title, IconButton, Photo, Paragraph, IconCloseButton } from '../atoms';
  import { configuration } from '../contexts/configuration';
  import { Elements } from '../contexts/configuration/types';
  import { goToPrevStep } from '../contexts/navigation';
  import { DocumentType, getDocImage, appState } from '../contexts/app-state';
  import { NavigationButtons } from '../molecules';
  import { documents, currentStepId, selectedDocumentInfo } from '../contexts/app-state/stores';
  import { preloadNextStepByCurrent } from '../services/preload-service';
  import {
    EActionNames,
    sendButtonClickEvent,
    EVerificationStatuses,
  } from '../utils/event-service';
  import { getLayoutStyles, getStepConfiguration, uiPack } from '../ui-packs';
  import { getFlowConfig } from '../contexts/flows/hooks';

  export let stepId;

  const step = getStepConfiguration($configuration, $uiPack, stepId);
  const flow = getFlowConfig($configuration);
  const style = getLayoutStyles($configuration, $uiPack, step);

  const stepNamespace = step.namespace!;
  const documentType =
    (($configuration.steps && $configuration.steps[$currentStepId].type) as DocumentType || $uiPack.steps[$currentStepId].type as DocumentType) || $selectedDocumentInfo.type;


  let image = '';
  let skipBackSide = false;

  $: {
    if (!documentType) {
      goToPrevStep(currentStepId, $configuration, $currentStepId);
    }
    if (documentType) {
      image = getDocImage(documentType, $documents);
    }
    if ($selectedDocumentInfo && !$selectedDocumentInfo.backSide) {
      skipBackSide = true;
    }
    preloadNextStepByCurrent(
      $configuration,
      configuration,
      $currentStepId,
      $uiPack,
      skipBackSide ? 'back-side' : undefined,
    );
  }
</script>

<div class="container" {style}>
  {#each step.elements as element}
    {#if element.type === Elements.IconButton}
      <IconButton
        configuration={element.props}
        on:click={() => goToPrevStep(currentStepId, $configuration, $currentStepId)}
      />
    {/if}
    {#if element.type === Elements.IconCloseButton && flow.showCloseButton}
      <IconCloseButton
        configuration={element.props}
        on:click={() => {
          sendButtonClickEvent(
            EActionNames.CLOSE,
            { status: EVerificationStatuses.DATA_COLLECTION },
            $appState,
            true,
          );
        }}
      />
    {/if}
    {#if element.type === Elements.Title}
      <Title configuration={element.props}>
        <T key={'title'} namespace={stepNamespace} />
      </Title>
    {/if}
    {#if element.type === Elements.Paragraph}
      <Paragraph configuration={element.props}>
        <T key={'description'} namespace={stepNamespace} />
      </Paragraph>
    {/if}
    {#if element.type === Elements.Photo}
      <Photo configuration={element.props} src={image} />
    {/if}
  {/each}
  <NavigationButtons {skipBackSide} />
</div>

<style>
  .container {
    padding: var(--padding);
    position: var(--position);
    background: var(--background);
    text-align: center;
    height: 100%;
    display: flex;
    flex-direction: column;
  }
</style>
