/*
  This file is part of CanFestival, a library implementing CanOpen
  Stack.

  Copyright (C): Edouard TISSERANT and Francis DUPIN

  See COPYING file for copyrights details.

  This library is free software; you can redistribute it and/or
  modify it under the terms of the GNU Lesser General Public
  License as published by the Free Software Foundation; either
  version 2.1 of the License, or (at your option) any later version.

  This library is distributed in the hope that it will be useful,
  but WITHOUT ANY WARRANTY; without even the implied warranty of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
  Lesser General Public License for more details.

  You should have received a copy of the GNU Lesser General Public
  License along with this library; if not, write to the Free Software
  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307
  USA
*/
/*!
** @file   objacces.c
** @author Edouard TISSERANT and Francis DUPIN
** @date   Tue Jun  5 08:55:23 2007
**
** @brief
**
**
*/

//#define DEBUG_WAR_CONSOLE_ON
//#define DEBUG_ERR_CONSOLE_ON

#include "data.h"

//We need the function implementation for linking
//Only a placeholder with a define isnt enough!
UNS8 accessDictionaryError(UNS16 index, UNS8 subIndex,
                           UNS32 sizeDataDict, UNS32 sizeDataGiven, UNS32 code)
{
  // sizeDataDict and sizeDataGiven only when OD_LENGTH_DATA_INVALID 

#ifdef DEBUG_WAR_CONSOLE_ON
  MSG_WAR(0x2B09,"Dictionary index : ", index);
  MSG_WAR(0X2B10,"           subindex : ", subIndex);
  switch (code) {
  case  OD_NO_SUCH_OBJECT:
    MSG_WAR(0x2B11,"Index not found ", sizeDataGiven);
    break;
  case OD_NO_SUCH_SUBINDEX :
    MSG_WAR(0x2B12,"SubIndex not found ", sizeDataGiven);
    break;
  case OD_WRITE_NOT_ALLOWED :
    MSG_WAR(0x2B13,"Write not allowed, data is read only ", index);
    break;
  case OD_LENGTH_DATA_INVALID :
    MSG_WAR(0x2B14,"Conflict size data. Should be (bytes) : ", sizeDataDict);
    MSG_WAR(0x2B15,"But you have given the size : ", sizeDataGiven);
    break;
  case OD_NOT_MAPPABLE :
    MSG_WAR(0x2B16,"Not mappable data in a PDO at index : ", index);
    break;
  case OD_NBR_OBJS_EXCEED_PDO_LEN :
    MSG_WAR(0x2B16,"Number and length of the objects to be mapped would exceed PDO length : ", sizeDataGiven);
    break;
  case OD_VALUE_TOO_LOW :
    MSG_WAR(0x2B17,"Value range error : value too low. SDOabort : ", sizeDataGiven);
    break;
  case OD_VALUE_TOO_HIGH :
    MSG_WAR(0x2B18,"Value range error : value too high. SDOabort : ", sizeDataGiven);
    MSG_WAR(0x2B18,"Expected max : ", sizeDataDict);
    break;
  case OD_READ_NOT_ALLOWED:
	  MSG_WAR(0x2B19,"Read not allowed, data is write only", sizeDataGiven);
 	  break;
  case OD_ACCESS_NOT_SUPPORTED:
	  MSG_WAR(0x2C11,"Mapping not allowed, wrong mapping procedure #", sizeDataGiven);
  default :
    MSG_WAR(0x2B26, "Unknown error code : ", code);
  }
  #endif

  return 0;
}

UNS32 _getODentry( CO_Data* d,
                   UNS16 wIndex,
                   UNS8 bSubindex,
                   void * pDestData,
                   UNS32 * pExpectedSize,
                   UNS8 * pDataType,
                   UNS8 checkAccess,
                   UNS8 endianize)
{ /* DO NOT USE MSG_ERR because the macro may send a PDO -> infinite
    loop if it fails. */
  UNS32 errorCode;
  UNS32 szData;
  const indextable *ptrTable;
  ODCallback_t *Callback;

  ptrTable = (*d->scanIndexOD)(wIndex, &errorCode, &Callback);

  if (errorCode != OD_SUCCESSFUL)
    return errorCode;
  if ( ptrTable->bSubCount <= bSubindex ) {
    /* Subindex not found */
    accessDictionaryError(wIndex, bSubindex, 0, ptrTable->bSubCount, OD_NO_SUCH_SUBINDEX);
    return OD_NO_SUCH_SUBINDEX;
  }

  if (checkAccess && (ptrTable->pSubindex[bSubindex].bAccessType & WO)) {
    MSG_WAR(0x2B30, "Access Type : ", ptrTable->pSubindex[bSubindex].bAccessType);
    accessDictionaryError(wIndex, bSubindex, 0, 0, OD_READ_NOT_ALLOWED);
    return OD_READ_NOT_ALLOWED;
  }

  *pDataType = ptrTable->pSubindex[bSubindex].bDataType;
  szData = ptrTable->pSubindex[bSubindex].size;

  if (*pExpectedSize == 0 ||
     *pExpectedSize == szData ||
     /* allow to fetch a shorter string than expected */
     (*pDataType >= visible_string && *pExpectedSize < szData)) 
  { 

#ifdef CANOPEN_BIG_ENDIAN
#warning CANOPEN_BIG_ENDIAN is set!
     if (endianize && *pDataType > boolean && !(
            *pDataType >= visible_string && 
            *pDataType <= domain)) {
      /* data must be transmited with low byte first */
      UNS8 i, j = 0;
      MSG_WAR(boolean, "data type ", *pDataType);
      MSG_WAR(visible_string, "data type ", *pDataType);
      for ( i = szData ; i > 0 ; i--) {
        MSG_WAR(i," ", j);
        ((UNS8*)pDestData)[j++] =
          ((UNS8*)ptrTable->pSubindex[bSubindex].pObject)[i-1];
      }
      *pExpectedSize = szData;
    }
    else /* no endianisation change */
#endif
    if (*pDataType != visible_string) {
        memcpy(pDestData, ptrTable->pSubindex[bSubindex].pObject,szData);
        *pExpectedSize = szData;
    }else{
        /* TODO : CONFORM TO DS-301 : 
         *  - stop using NULL terminated strings
         *  - store string size in td_subindex 
         * */
        /* Copy null terminated string to user, and return discovered size */
        UNS8 *ptr = (UNS8*)ptrTable->pSubindex[bSubindex].pObject;
        UNS8 *ptr_start = ptr;
        /* *pExpectedSize IS < szData . if null, use szData */
        UNS8 *ptr_end = ptr + (*pExpectedSize ? *pExpectedSize : szData) ; 
        UNS8 *ptr_dest = (UNS8*)pDestData;

        while( *ptr && ptr < ptr_end){
            *(ptr_dest++) = *(ptr++);
        } 

        *pExpectedSize = (UNS32) (ptr - ptr_start);
        /* terminate string if not maximum length */
        if (*pExpectedSize < szData)                // this is a bug, as it destroys the dictionary entry by truncating it!!!
            *(ptr) = 0; 
    }

    return OD_SUCCESSFUL;
  }
  else 
  { /* Error ! */
    accessDictionaryError(wIndex, bSubindex, szData, *pExpectedSize, OD_LENGTH_DATA_INVALID);
    *pExpectedSize = szData;
    return OD_LENGTH_DATA_INVALID;
  }
}

UNS32 _setODentry( CO_Data* d,
                   UNS16 wIndex,
                   UNS8 bSubindex,
                   void * pSourceData,
                   UNS32 * pExpectedSize,
                   UNS8 checkAccess,
                   UNS8 endianize)
{
    UNS8 const maxMapObj = 8;

    UNS32 szData;
    UNS8 dataType;
    UNS32 data32;
    UNS16 data16;
    UNS16 odIndex_ComParm = 0;
    UNS16 odIndex_MapParm = 0;
    UNS32 errorCode;
    UNS16 com_type;
    const indextable *ptrTable;			/* Index table of the object to be searched */
    const indextable *ptrTable_ComParm;	/* Index table of the according communication parameters if the object is RPDO/TPDO */
    const indextable *ptrTable_MapParm;	/* Index table of the according mapping parameters if the object is RPDO/TPDO */
    const indextable *ptrTable_obj;		/* Index table of the mapped object into the PDO mapping table */

    ODCallback_t *Callback;
    ODCallback_t *Callback_ComParm;
    ODCallback_t *Callback_MapParm;
    ODCallback_t *Callback_obj;

    enum _com_type {RPDO=1, TPDO};
    /* fj: generalized!                              actually, it is only to check if 'Mapping not allowed'*/
    if (wIndex >= 0x1600 && wIndex <= 0x17ff ||
        wIndex >= 0x1A00 && wIndex <= 0x1bff)
    {
        odIndex_ComParm = wIndex - 0x0200;
        odIndex_MapParm = wIndex;
        com_type = wIndex <= 0x17ff? RPDO : TPDO;
    }
    else
    {
        com_type = 0;
    }

    ptrTable =(*d->scanIndexOD)(wIndex, &errorCode, &Callback);
    if (errorCode != OD_SUCCESSFUL) {
        return errorCode;
    }

    if (com_type == TPDO ||com_type == RPDO )
    {
        ptrTable_ComParm =(*d->scanIndexOD)(odIndex_ComParm, &errorCode, &Callback_ComParm);
        if (errorCode != OD_SUCCESSFUL) {
            return errorCode;
        }
        ptrTable_MapParm =(*d->scanIndexOD)(odIndex_MapParm, &errorCode, &Callback_MapParm);
        if (errorCode != OD_SUCCESSFUL) {
            return errorCode;
        }
    }
    else
    {  /* To avoid compiler warnings */
        ptrTable_ComParm =(*d->scanIndexOD)(wIndex, &errorCode, &Callback);
        ptrTable_MapParm =(*d->scanIndexOD)(wIndex, &errorCode, &Callback);
    }

    if (ptrTable->bSubCount <= bSubindex)
    {
        /* Subindex not found */
        accessDictionaryError(wIndex, bSubindex, 0, ptrTable->bSubCount, OD_NO_SUCH_OBJECT);
        return OD_NO_SUCH_OBJECT; /*OD_NO_SUCH_SUBINDEX */
    }

    if (checkAccess &&
        (ptrTable->pSubindex[bSubindex].bAccessType & RO) &&
        (d->nodeState == Operational || d->nodeState == Pre_operational))
    {
        MSG_WAR(0x2B25, "Access Type : ", ptrTable->pSubindex[bSubindex].bAccessType);
        accessDictionaryError(wIndex, bSubindex, 0, 0, OD_WRITE_NOT_ALLOWED);
        return OD_WRITE_NOT_ALLOWED;
    }

    /* Start: Check PDO Mapping Procedure */
    if ((com_type == TPDO || com_type == RPDO) &&
        bSubindex == 0 &&
        (d->nodeState == Operational || d->nodeState == Pre_operational))
    {
        if (!(*(UNS32*)ptrTable_ComParm->pSubindex[1].pObject & 0x80000000))
        {
            /* Wrong mapping procedure: Mapping not allowed: PDO COB-ID still valid! */
            accessDictionaryError(wIndex, bSubindex, 0, 1, OD_ACCESS_NOT_SUPPORTED);
            return 0x08000000; /* OD_ACCESS_NOT_SUPPORTED; */
        }
        
        data32 = *(UNS8*)pSourceData;     // subindex 0 is UNSIGNED8 always and maxMapObj is also UNS8
        if (data32 > maxMapObj) /* hard coded */
        {
            /* mapping to many objects. Test command 0x2F001A0009000000*/
            MSG_WAR(0x2B21, "Mapping to many objects : ", data32);
            accessDictionaryError(wIndex, bSubindex, maxMapObj, data32, OD_VALUE_TOO_HIGH);
            return OD_VALUE_TOO_HIGH;
        }

        if (wIndex >= 0x1600 && wIndex <= 0x16ff ||
            wIndex >= 0x1a00 && wIndex <= 0x1aff)   /* bSubindex is 0 here */
        {
            /* check if trying to map more than 64 bits */
            UNS8 i;
            UNS8 cnt = 0;
            for (i=1; i<=data32; i++)
            {
                cnt += *(UNS8 *)ptrTable_MapParm->pSubindex[i].pObject;
            }
            if (cnt > 64)
            {
                MSG_WAR(0x2B21, "Mapping to more than 64 bits : ", cnt);
                accessDictionaryError(wIndex, bSubindex, 0, cnt, OD_NBR_OBJS_EXCEED_PDO_LEN);
                return OD_NBR_OBJS_EXCEED_PDO_LEN;
            }
        }
    }

    if ((com_type == TPDO || com_type == RPDO) &&
        bSubindex > 0 &&
        (d->nodeState == Operational || d->nodeState == Pre_operational))
    {
        if (*(UNS8*) ptrTable_MapParm->pSubindex[0].pObject != 0 ||
            !(*(UNS32*) ptrTable_ComParm->pSubindex[1].pObject & 0x80000000))
        {
            /* Wrong mapping procedure: Mapping not allowed! */
            accessDictionaryError(wIndex, bSubindex, 0, 2, OD_ACCESS_NOT_SUPPORTED);
            return OD_ACCESS_NOT_SUPPORTED; /*0x08000000 */
        }
        else
        {
            /* Correct mapping procedure; Mapping allowed! */
            data32 = *(UNS32*)pSourceData;
            data16 = data32 >> 16;

            ptrTable_obj = (*d->scanIndexOD)(data16, &errorCode, &Callback_obj);
            if (errorCode != OD_SUCCESSFUL)
            {
                if (data16 <= 0x7)
                {
                    /* Tried to map a dummy variable */
                    errorCode = OD_ACCESS_NOT_SUPPORTED;
                }
                
                accessDictionaryError(wIndex, bSubindex, 0, 3, errorCode);
                return errorCode;
            }
            
            if (bSubindex > maxMapObj) /* hard coded! */
            {
                /* mapping too long */
                MSG_WAR(0x2B21, "Mapping to many objects, SubIdx : ", bSubindex);
                accessDictionaryError(wIndex, bSubindex, maxMapObj, bSubindex, OD_VALUE_TOO_HIGH);
                return OD_VALUE_TOO_HIGH;
            }
            
            if (!(ptrTable_obj->pSubindex[0].bAccessType & MAPPABLE))
            {
                /* Object not mappable*/
                accessDictionaryError(wIndex, bSubindex, 0, 0, OD_NOT_MAPPABLE);
                return OD_NOT_MAPPABLE;
            }
            
            if (com_type == RPDO && !(ptrTable_obj->pSubindex[0].bAccessType & WO))
            {
                /* RPDO must be WO */
                MSG_WAR(0x2B21, "Access Type : ", ptrTable_obj->pSubindex[0].bAccessType);
                accessDictionaryError(wIndex, bSubindex, 0, 100, OD_NO_SUCH_OBJECT);
                return OD_NO_SUCH_OBJECT;
            }
            
            if (com_type == TPDO && !(ptrTable_obj->pSubindex[0].bAccessType & RO))
            {
                /* TPDO must be RO */
                MSG_WAR(0x2B21, "Access Type : ", ptrTable_obj->pSubindex[0].bAccessType);
                accessDictionaryError(wIndex, bSubindex, 0, 101, OD_NO_SUCH_OBJECT);
                return OD_NO_SUCH_OBJECT;
            }
        }
    }
    /* End: Check PDO Mapping Procedure */
    
    dataType = ptrTable->pSubindex[bSubindex].bDataType;
    szData = ptrTable->pSubindex[bSubindex].size;

    if (*pExpectedSize == 0 ||
        *pExpectedSize == szData ||
        (dataType == visible_string && *pExpectedSize < szData))    /* allow to store a shorter string than entry size */
    {
#ifdef CANOPEN_BIG_ENDIAN
        /* re-endianize do not occur for bool, strings time and domains */
        if (endianize && 
            dataType > boolean && 
            !(dataType >= visible_string && dataType <= domain))
        {
            /* we invert the data source directly. This let us do range testing without */
            /* additional temp variable */
            UNS8 i;
            for ( i = 0; i < ( ptrTable->pSubindex[bSubindex].size >> 1); i++)
            {
                UNS8 tmp =((UNS8 *)pSourceData) [(ptrTable->pSubindex[bSubindex].size - 1) - i];
                ((UNS8 *)pSourceData) [(ptrTable->pSubindex[bSubindex].size - 1) - i] = ((UNS8 *)pSourceData)[i];
                ((UNS8 *)pSourceData)[i] = tmp;
            }
        }
#endif
        errorCode = (*d->valueRangeTest)(dataType, pSourceData);
        if (errorCode == OD_SUCCESSFUL)
        {
            // check PDO COB-ID limits
            if ((wIndex >= 0x1400 && wIndex <= 0x15ff ||
                 wIndex >= 0x1800 && wIndex <= 0x19ff) &&
                bSubindex == 1)
            {
                // CAN id cannot be 0 (CT)
                if (((*(UNS32*)pSourceData) & 0x1fffffff) == 0)
                errorCode = OD_VALUE_TOO_LOW;
            }
        }
        
        if (errorCode) {
            accessDictionaryError(wIndex, bSubindex, 0, *(UNS32*)pSourceData, errorCode);
            return errorCode;
        }

        memcpy(ptrTable->pSubindex[bSubindex].pObject, pSourceData, *pExpectedSize);

        /* TODO : CONFORM TO DS-301 :
        *  - stop using NULL terminated strings
        *  - store string size in td_subindex
        * */

        /* terminate visible_string with '\0' */
        if (dataType == visible_string && *pExpectedSize < szData)
            ((UNS8*)ptrTable->pSubindex[bSubindex].pObject)[*pExpectedSize] = 0;

        *pExpectedSize = szData;

        /* Callbacks */
        if (Callback && Callback[bSubindex])
        {
            errorCode = (Callback[bSubindex])(d, ptrTable, bSubindex);

            if (errorCode != OD_SUCCESSFUL)
            {
                return errorCode;
            }
        }

        /* TODO : Store dans NVRAM */
        if (ptrTable->pSubindex[bSubindex].bAccessType & TO_BE_SAVE){
            (*d->storeODSubIndex)(d, wIndex, bSubindex);
        }
        return OD_SUCCESSFUL;
    }
    else
    {
        accessDictionaryError(wIndex, bSubindex, szData, *pExpectedSize, OD_LENGTH_DATA_INVALID);
        *pExpectedSize = szData;
        return OD_LENGTH_DATA_INVALID;
    }
}

const indextable * scanIndexOD (CO_Data* d, UNS16 wIndex, UNS32 *errorCode, ODCallback_t **Callback)
{
    return (*d->scanIndexOD)(wIndex, errorCode, Callback);
}

UNS32 RegisterSetODentryCallBack(CO_Data* d, UNS16 wIndex, UNS8 bSubindex, ODCallback_t Callback)
{
UNS32 errorCode;
ODCallback_t *CallbackList;
const indextable *odentry;

  odentry = scanIndexOD (d, wIndex, &errorCode, &CallbackList);
  if (errorCode == OD_SUCCESSFUL  &&  CallbackList  &&  bSubindex < odentry->bSubCount) 
    CallbackList[bSubindex] = Callback;
  return errorCode;
}

void _storeODSubIndex (CO_Data* d, UNS16 wIndex, UNS8 bSubindex){}
